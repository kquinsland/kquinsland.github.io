Not a super big fan of having to have `npm` and similar installed on system.

There are a few people out there that have put the resume tool into a docker container.


```shell
❯ docker run -it --rm -p 4000:4000 -v /home/karl/Documents/blog/static/resume:/opt/resume olbat/jsonresume bash
```

Once inside the container, will need to actually install the resume-cli tool

```shell
node@5914c6eb9069:~$ sudo npm install -g --unsafe-perm=true resume-cli
<...>
```

Various resume themes can be installed, too.
See: https://jsonresume.org/themes/

```shell
node@d676170b1ed0:~$ sudo npm install -g --unsafe-perm=true  jsonresume-theme-spartan
+ jsonresume-theme-spartan@0.3.0
<...>
```

Then run the web server to track changes in real time:

```shell
node@d676170b1ed0:~$ resume serve -r /opt/resume/kq.json
```

View the resume http://localhost:4000/


And when ready to export to a file:

```shell
node@d676170b1ed0:/opt/resume$ resume export -r kq.json --format html --theme /usr/lib/node_modules/jsonresume-theme-flat/ kq.html
```
