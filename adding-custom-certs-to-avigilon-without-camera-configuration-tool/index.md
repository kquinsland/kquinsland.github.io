# Loading a certificate onto Avigilon without using CCT

Automating certificate deployment on Avigilon cameras without using the Camera Configuration Tool (CCT).

Canonical: https://karlquinsland.com/adding-custom-certs-to-avigilon-without-camera-configuration-tool/
Published: 2026-06-01
Updated: 2026-06-01
Tags: avigilon, onvif, letsencrypt, certificates, automation


---

# Loading a certificate onto Avigilon without using CCT

I recently won an auction for a used Avigilon camera with the intention of replacing my cheap Reolink cameras with something a bit more "pro" and that would integrate better with Home Assistant.

This isn't going to be a "review" of the camera or an analysis of the firmware, just a quick "I googled, found nothing so now I'm sharing how I did it for the next guy" post.

## The Problem

Out of the box, the camera ships with a self-signed certificate.
The web UI allows you to switch between OpenSSL or FIPS-140 compliant "encryption engine" but it does not allow you to load your own certificate.

![Yes, really. This is all that you can adjust from the web UI. No option to upload your own cert.](https://karlquinsland.com/adding-custom-certs-to-avigilon-without-camera-configuration-tool/images/img01.webp)

_Yes, really. This is all that you can adjust from the web UI. No option to upload your own cert._


You can customize a lot of _other_ things about the camera from the web UI but the web-server certificates is not one of them... for some reason.
The web UI lets you manage 802.1X certificates but ... not the web server certificates.

WHY?

You can re-use the same html form/input from the 802.1X certificate management page to upload your own certs for the web server but ... they just don't give you that option.

Docs indicate that at least [_newer_ cameras can do this through the web UI](https://docs.avigilon.com/bundle/pro-camera-web-interface/page/network-and-security/create-certificates.htm) embedded into the camera... but this guy just isn't going to get that update.

The "fall back" is to use their [Camera Configuration Tool (CCT)](https://docs.avigilon.com/bundle/camera-configuration-tool-2-16/page/CCT-Intro.htm) which is a Windows only application.
You also need to create an account to download the tool.

No thank you.

I am trying to _automate_ updating certificates on these cameras and "automate" in this case means "write a script that I can run in some container on a Linux host" and the CCT doesn't really fit into that workflow.

## The solution

Because CCT is a `.net` application, it was _trivial_ to reverse engineer the protocol details.

As it turns out, the CCT app only exposes graphical elements for working with camera-generated CSRs and uploading only the signed certificate.

Fortunately, the API that the CCT uses under the hood is the [ONVIF Device Management](https://www.onvif.org/ver10/device/wsdl/devicemgmt.wsdl) API which lets us do a _lot_ more than just upload a signed certificate; the `LoadCertificateWithPrivateKey` command is not exposed via CCT but it is supported by the camera firmware!

After a bit more testing with a sample LetsEncrypt certificate, I was able to confirm that the camera will accept a custom certificate and private key without using CCT at all:

- A Let's Encrypt leaf certificate can be uploaded as DER X.509.
  - The matching private key can be uploaded as unencrypted DER PKCS#8.
- The new certificate is not useful until it is activated with `SetCertificatesStatus`.

One other thing to call out from testing:

- Uploading CA certificates to the camera CA store did not make the camera serve a full HTTPS chain! This means that you can/should only upload the leaf certificates. Fortunately most modern browsers will be able to handle this just fine as long as the leaf certificate is valid and signed by a trusted CA.

### Show Me The Code

This is a lightly modified version of the script I wrote while testing things.
It's a bit more verbose than I would like but:

- I wanted to make it explicit and robust for automation purposes
- I wanted to have zero dependencies outside of `cryptography` and `requests` from `stdlib` so this would run cheaply anywhere [`uv`](https://docs.astral.sh/uv/) was present

And also SOAP is just ... grossly verbose. Nothing I could do about that!

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "cryptography>=45.0.0",
#   "requests>=2.32.0",
# ]
# ///
from __future__ import annotations

import argparse
import base64
import dataclasses
import hashlib
import os
import re
import socket
import ssl
import sys
import textwrap
import time
import xml.etree.ElementTree as ET
from collections.abc import Iterator, Sequence
from datetime import UTC, datetime
from pathlib import Path
from typing import Literal
from xml.sax.saxutils import escape

import requests
from cryptography import x509
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.serialization import load_pem_private_key
from cryptography.x509.oid import ExtensionOID, NameOID
from requests.auth import HTTPDigestAuth

SOAP_ENV_NS = "http://www.w3.org/2003/05/soap-envelope"
TDS_NS = "http://www.onvif.org/ver10/device/wsdl"
TT_NS = "http://www.onvif.org/ver10/schema"
WSSE_NS = (
    "http://docs.oasis-open.org/wss/2004/01/"
    "oasis-200401-wss-wssecurity-secext-1.0.xsd"
)
WSU_NS = (
    "http://docs.oasis-open.org/wss/2004/01/"
    "oasis-200401-wss-wssecurity-utility-1.0.xsd"
)
PASSWORD_DIGEST_TYPE = (
    "http://docs.oasis-open.org/wss/2004/01/"
    "oasis-200401-wss-username-token-profile-1.0#PasswordDigest"
)

AuthMode = Literal["wsse", "digest"]


class Error(Exception):
    pass


@dataclasses.dataclass(frozen=True)
class CameraCert:
    cert_id: str
    fingerprint: str | None


@dataclasses.dataclass(frozen=True)
class CertStatus:
    cert_id: str
    active: bool


@dataclasses.dataclass(frozen=True)
class LocalMaterial:
    cert_der: bytes
    key_der_pkcs8: bytes
    fingerprint: str
    not_after: datetime
    tls_name: str | None


def main(argv: Sequence[str] | None = None) -> int:
    args = parse_args(argv)
    password = args.password or os.environ.get("AVIGILON_PASSWORD")
    if not password:
        raise Error("Pass --password or set AVIGILON_PASSWORD")

    material = load_material(args.cert, args.key, args.host)
    cert_id = args.certificate_id or (
        f"le-{material.not_after:%Y%m%d}-{material.fingerprint[:12]}"
    )
    endpoint = endpoint_url(args.scheme, args.host, args.port, args.path)
    auth_mode: AuthMode = args.auth

    print(f"endpoint: {endpoint}")
    print(f"target certificate id: {cert_id}")
    print(f"target fingerprint: {material.fingerprint}")

    soap_call(
        endpoint,
        args.user,
        password,
        auth_mode,
        "GetDeviceInformation",
        "<tds:GetDeviceInformation/>",
        args.timeout,
        args.verify_onvif_tls,
    )

    certs = get_certificates(
        endpoint, args.user, password, auth_mode, args.timeout, args.verify_onvif_tls
    )
    statuses = get_statuses(
        endpoint, args.user, password, auth_mode, args.timeout, args.verify_onvif_tls
    )
    active_id = next((status.cert_id for status in statuses if status.active), None)
    active_cert = next((cert for cert in certs if cert.cert_id == active_id), None)

    if active_cert and active_cert.fingerprint == material.fingerprint:
        print("camera is already serving this certificate")
        verify_served(args.host, args.https_port, material.tls_name or args.host, material)
        return 0

    existing_match = next(
        (cert.cert_id for cert in certs if cert.fingerprint == material.fingerprint),
        None,
    )
    target_id = existing_match or cert_id

    conflicting = next(
        (
            cert
            for cert in certs
            if cert.cert_id == cert_id and cert.fingerprint != material.fingerprint
        ),
        None,
    )
    if conflicting:
        raise Error(f"certificate id already exists with different bytes: {cert_id}")

    if not existing_match:
        print("uploading certificate and private key")
        load_certificate_with_private_key(
            endpoint,
            args.user,
            password,
            auth_mode,
            target_id,
            material,
            args.timeout,
            args.verify_onvif_tls,
        )
    else:
        print(f"certificate already exists on camera as {target_id}; activating")

    all_ids = unique([target_id, *(cert.cert_id for cert in certs)])
    set_statuses(
        endpoint,
        args.user,
        password,
        auth_mode,
        [CertStatus(item, item == target_id) for item in all_ids],
        args.timeout,
        args.verify_onvif_tls,
    )

    verify_served(args.host, args.https_port, material.tls_name or args.host, material)
    print("deploy complete")
    return 0


def parse_args(argv: Sequence[str] | None) -> argparse.Namespace:
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", required=True)
    parser.add_argument("--user", required=True)
    parser.add_argument("--password")
    parser.add_argument("--cert", type=Path, required=True)
    parser.add_argument("--key", type=Path, required=True)
    parser.add_argument("--certificate-id")
    parser.add_argument("--scheme", choices=["http", "https"], default="http")
    parser.add_argument("--port", type=int)
    parser.add_argument("--path", default="/onvif/device_service")
    parser.add_argument("--auth", choices=["wsse", "digest"], default="wsse")
    parser.add_argument("--timeout", type=float, default=20.0)
    parser.add_argument("--https-port", type=int, default=443)
    parser.add_argument("--verify-onvif-tls", action="store_true")
    return parser.parse_args(argv)


def endpoint_url(scheme: str, host: str, port: int | None, path: str) -> str:
    default_port = 443 if scheme == "https" else 80
    port_part = "" if port is None or port == default_port else f":{port}"
    path = path if path.startswith("/") else f"/{path}"
    return f"{scheme}://{host}{port_part}{path}"


def load_material(cert_path: Path, key_path: Path, fallback_tls_name: str) -> LocalMaterial:
    cert_blocks = extract_pem_blocks(cert_path.read_bytes(), "CERTIFICATE")
    if not cert_blocks:
        raise Error(f"no certificate PEM block found in {cert_path}")
    leaf = x509.load_pem_x509_certificate(cert_blocks[0])
    key = load_pem_private_key(key_path.read_bytes(), password=None)

    cert_public = leaf.public_key().public_bytes(
        serialization.Encoding.DER,
        serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    key_public = key.public_key().public_bytes(
        serialization.Encoding.DER,
        serialization.PublicFormat.SubjectPublicKeyInfo,
    )
    if cert_public != key_public:
        raise Error("certificate and private key do not match")

    cert_der = leaf.public_bytes(serialization.Encoding.DER)
    key_der = key.private_bytes(
        serialization.Encoding.DER,
        serialization.PrivateFormat.PKCS8,
        serialization.NoEncryption(),
    )
    return LocalMaterial(
        cert_der=cert_der,
        key_der_pkcs8=key_der,
        fingerprint=hashlib.sha256(cert_der).hexdigest(),
        not_after=leaf.not_valid_after_utc,
        tls_name=preferred_tls_name(leaf) or fallback_tls_name,
    )


def get_certificates(
    endpoint: str,
    user: str,
    password: str,
    auth_mode: AuthMode,
    timeout: float,
    verify_tls: bool,
) -> list[CameraCert]:
    root = soap_call(
        endpoint,
        user,
        password,
        auth_mode,
        "GetCertificates",
        "<tds:GetCertificates/>",
        timeout,
        verify_tls,
    )
    result: list[CameraCert] = []
    for node in iter_local(root, {"NvtCertificate", "NVTCertificate"}):
        cert_id = child_text(node, "CertificateID")
        data = descendant_text(node, "Data")
        if cert_id:
            result.append(CameraCert(cert_id, certificate_data_fingerprint(data)))
    return result


def get_statuses(
    endpoint: str,
    user: str,
    password: str,
    auth_mode: AuthMode,
    timeout: float,
    verify_tls: bool,
) -> list[CertStatus]:
    root = soap_call(
        endpoint,
        user,
        password,
        auth_mode,
        "GetCertificatesStatus",
        "<tds:GetCertificatesStatus/>",
        timeout,
        verify_tls,
    )
    result: list[CertStatus] = []
    for node in iter_local(root, {"CertificateStatus"}):
        cert_id = child_text(node, "CertificateID")
        active = child_text(node, "Status")
        if cert_id and active is not None:
            result.append(CertStatus(cert_id, active.lower() in {"true", "1"}))
    return result


def load_certificate_with_private_key(
    endpoint: str,
    user: str,
    password: str,
    auth_mode: AuthMode,
    cert_id: str,
    material: LocalMaterial,
    timeout: float,
    verify_tls: bool,
) -> None:
    body = textwrap.dedent(
        f"""\
        <tds:LoadCertificateWithPrivateKey>
          <tds:CertificateWithPrivateKey>
            <tt:CertificateID>{xml_text(cert_id)}</tt:CertificateID>
            <tt:Certificate>
              <tt:Data>{base64.b64encode(material.cert_der).decode("ascii")}</tt:Data>
            </tt:Certificate>
            <tt:PrivateKey>
              <tt:Data>{base64.b64encode(material.key_der_pkcs8).decode("ascii")}</tt:Data>
            </tt:PrivateKey>
          </tds:CertificateWithPrivateKey>
        </tds:LoadCertificateWithPrivateKey>
        """
    )
    soap_call(endpoint, user, password, auth_mode, "LoadCertificateWithPrivateKey", body, timeout, verify_tls)


def set_statuses(
    endpoint: str,
    user: str,
    password: str,
    auth_mode: AuthMode,
    statuses: Sequence[CertStatus],
    timeout: float,
    verify_tls: bool,
) -> None:
    items = "".join(
        "<tds:CertificateStatus>"
        f"<tt:CertificateID>{xml_text(status.cert_id)}</tt:CertificateID>"
        f"<tt:Status>{'true' if status.active else 'false'}</tt:Status>"
        "</tds:CertificateStatus>"
        for status in statuses
    )
    soap_call(
        endpoint,
        user,
        password,
        auth_mode,
        "SetCertificatesStatus",
        f"<tds:SetCertificatesStatus>{items}</tds:SetCertificatesStatus>",
        timeout,
        verify_tls,
    )


def soap_call(
    endpoint: str,
    user: str,
    password: str,
    auth_mode: AuthMode,
    operation: str,
    body_xml: str,
    timeout: float,
    verify_tls: bool,
) -> ET.Element:
    action = f"{TDS_NS}/{operation}"
    header_xml = wsse_header(user, password) if auth_mode == "wsse" else ""
    envelope = textwrap.dedent(
        f"""\
        <?xml version="1.0" encoding="utf-8"?>
        <s:Envelope xmlns:s="{SOAP_ENV_NS}" xmlns:tds="{TDS_NS}" xmlns:tt="{TT_NS}">
          <s:Header>{header_xml}</s:Header>
          <s:Body>{body_xml}</s:Body>
        </s:Envelope>
        """
    )
    auth = HTTPDigestAuth(user, password) if auth_mode == "digest" else None
    response = requests.post(
        endpoint,
        data=envelope.encode("utf-8"),
        headers={
            "Content-Type": f'application/soap+xml; charset=utf-8; action="{action}"',
            "SOAPAction": f'"{action}"',
            "Accept": "application/soap+xml, text/xml, */*",
        },
        auth=auth,
        timeout=timeout,
        verify=verify_tls,
    )

    try:
        root = ET.fromstring(response.content)
    except ET.ParseError as exc:
        snippet = response.text[:1000].replace("\n", " ")
        raise Error(f"{operation}: HTTP {response.status_code}: {snippet}") from exc

    fault = soap_fault_text(root)
    if fault:
        raise Error(f"{operation}: HTTP {response.status_code}: {fault}")
    if not 200 <= response.status_code < 300:
        snippet = response.text[:1000].replace("\n", " ")
        raise Error(f"{operation}: HTTP {response.status_code}: {snippet}")
    return root


def wsse_header(user: str, password: str) -> str:
    nonce = os.urandom(16)
    created = datetime.now(UTC).strftime("%Y-%m-%dT%H:%M:%SZ")
    digest = hashlib.sha1(nonce + created.encode() + password.encode()).digest()
    return textwrap.dedent(
        f"""\
        <wsse:Security xmlns:wsse="{WSSE_NS}">
          <wsse:UsernameToken xmlns:wsu="{WSU_NS}" wsu:Id="SecurityToken-{os.urandom(16).hex()}">
            <wsse:Username>{xml_text(user)}</wsse:Username>
            <wsse:Password Type="{PASSWORD_DIGEST_TYPE}">{base64.b64encode(digest).decode("ascii")}</wsse:Password>
            <wsse:Nonce>{base64.b64encode(nonce).decode("ascii")}</wsse:Nonce>
            <wsu:Created>{created}</wsu:Created>
          </wsse:UsernameToken>
        </wsse:Security>
        """
    )


def verify_served(host: str, port: int, server_name: str, material: LocalMaterial) -> None:
    last = ""
    for attempt in range(1, 9):
        try:
            context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
            context.check_hostname = False
            context.verify_mode = ssl.CERT_NONE
            with socket.create_connection((host, port), timeout=10) as sock:
                with context.wrap_socket(sock, server_hostname=server_name) as tls_sock:
                    cert_der = tls_sock.getpeercert(binary_form=True)
            if not cert_der:
                raise Error("TLS peer did not return a certificate")
            served = hashlib.sha256(cert_der).hexdigest()
            if served == material.fingerprint:
                print(f"verified served HTTPS leaf on attempt {attempt}")
                return
            last = f"served {served}, expected {material.fingerprint}"
        except OSError as exc:
            last = str(exc)
        time.sleep(2)
    raise Error(f"served certificate verification failed: {last}")


def certificate_data_fingerprint(data_text: str | None) -> str | None:
    if not data_text:
        return None
    try:
        raw = base64.b64decode("".join(data_text.split()), validate=True)
    except ValueError:
        return None
    try:
        cert = x509.load_der_x509_certificate(raw)
    except ValueError:
        try:
            blocks = extract_pem_blocks(raw, "CERTIFICATE")
            cert = x509.load_pem_x509_certificate(blocks[0])
        except (IndexError, ValueError):
            return None
    return hashlib.sha256(cert.public_bytes(serialization.Encoding.DER)).hexdigest()


def preferred_tls_name(cert: x509.Certificate) -> str | None:
    try:
        san = cert.extensions.get_extension_for_oid(ExtensionOID.SUBJECT_ALTERNATIVE_NAME)
        dns_names = san.value.get_values_for_type(x509.DNSName)
        if dns_names:
            return dns_names[0]
    except x509.ExtensionNotFound:
        pass
    names = cert.subject.get_attributes_for_oid(NameOID.COMMON_NAME)
    return names[0].value if names and isinstance(names[0].value, str) else None


def soap_fault_text(root: ET.Element) -> str | None:
    fault = next(iter_local(root, {"Fault"}), None)
    if fault is None:
        return None
    texts = [
        node.text.strip()
        for node in iter_local(fault, {"Text", "faultstring"})
        if node.text and node.text.strip()
    ]
    return "; ".join(texts) if texts else ET.tostring(fault, encoding="unicode")[:800]


def extract_pem_blocks(data: bytes, label: str) -> list[bytes]:
    pattern = re.compile(
        rb"-----BEGIN "
        + re.escape(label.encode("ascii"))
        + rb"-----.*?-----END "
        + re.escape(label.encode("ascii"))
        + rb"-----\s*",
        flags=re.DOTALL,
    )
    return [match.group(0) for match in pattern.finditer(data)]


def iter_local(root: ET.Element, names: set[str]) -> Iterator[ET.Element]:
    for node in root.iter():
        if node.tag.rsplit("}", 1)[-1] in names:
            yield node


def child_text(root: ET.Element, name: str) -> str | None:
    for child in list(root):
        if child.tag.rsplit("}", 1)[-1] == name and child.text:
            return child.text.strip()
    return None


def descendant_text(root: ET.Element, name: str) -> str | None:
    node = next(iter_local(root, {name}), None)
    return node.text.strip() if node is not None and node.text else None


def xml_text(value: str) -> str:
    return escape(value, {'"': "&quot;", "'": "&apos;"})


def unique(values: Sequence[str]) -> list[str]:
    result: list[str] = []
    for value in values:
        if value not in result:
            result.append(value)
    return result


if __name__ == "__main__":
    try:
        raise SystemExit(main())
    except Error as exc:
        print(f"error: {exc}", file=sys.stderr)
        raise SystemExit(2) from exc
```

