# Announcing The Missing ToDoist Tools

# TMTDT: The Missing ToDoist Tools 🎉

As the name implies, TMTDT started as a small collection of scripts that I used to augment ToDoist with features they can't/won't implement. It's grown quite a bit since then.

{{< figure name="demo gif" >}}

Those scripts started as simple idea and quickly morphed into a creaky, but essential, tool. As more features were added it continued too morph into an unmaintainable mess. Untangling that mess was on my todo list but never a high priority partly because of issues TMTDT was designed to solve 🤦.

I lost my day job right as the spring of 2020 [shelter in place](https://en.wikipedia.org/wiki/Stay-at-home_order#COVID-19_pandemic) order was put in place. TMTDT is certainly not how I _thought_ early summer was going to be spent, but a useful way nonetheless.

One significant chunk of my newly-freed time and a lengthy re-write later, I had something that looked promising. After a _lot_ more work and some tinkering, I've got something that feels like far more than a hacky tool for personal use.

## Why

Why not?

I had several scripts that I would run on a regular basis to do things like:

- set up projects with multiple different types of templates based on things like the date
- fix typos i routinely made while [quick adding](https://get.todoist.help/hc/en-us/articles/115001745265-Task-Quick-Add) tasks.
- re-schedule certain daily tasks if they had been missed

No joke, I'd spend a _lot_ of time fixing typos, expanding ideas for a project into proper sub-tasks and sections and other routine administrative work:

{{< figure name="admin_time_report" >}}

In that specific week, I kept track of how 37 hours were spent. Of those 37, about 4 were spent on general Inbox review / process. At least 3 of those 4 were spent _directly_ on task triage. 3/37 is about 8%. With no full time day job - [hire me]({{< relref "/contact" >}}) - that's a fairly representative breakdown for a given week; some time planning the things, lots of time doing the things.

What would you do with 8% of your time back each week?

I'm kidding.

We all know that I'm not getting any free time back from this.

{{< figure name="xkcd-1319" >}}

I've written this software to suit my needs, but the decision to make it a bit more robust and featured before release is because I've noticed a few 'new' tools to better augment ToDoist:

- [autodoist](https://github.com/Hoffelhas/autodoist)
- [shortcuts](https://chrome.google.com/webstore/detail/todoist-shortcuts-gmail-v/dehmghpdcahlffompjagejmgbcfahndp)
- [taskbutler](https://github.com/6uhrmittag/taskbutler)
- [planner](https://planner-todo.web.app/). This is more of a UI, but it's a **very** pretty app.

And I'm genuinely curious about how other people will user it. I am always appreciative when somebody finds a smart way to save some time and shares their trick. I'm hoping somebody figures out how to use this tool in a way that I can learn from.

An announcement post for TMTDT is now on [/r/todoist](https://old.reddit.com/r/todoist/comments/hyfl3o/introducing_the_missing_todoist_tools_a/).

## How

The code and some documentation can be found on [GitHub](https://github.com/kquinsland/the-missing-todoist-tools).

The demo gif at the top of this post illustrates a simple example, but more through (read: contrived) examples are included in the [jobs](https://github.com/kquinsland/the-missing-todoist-tools/tree/master/jobs/v1) directory.

The short version:

> Scriptable tools for building jobs to manage a TooDoist account

Slightly longer version:

> Architecture inspired by Elastic Search's [Curator](https://github.com/elastic/curator/tree/master/curator). Functionality built for ToDoist.

### Demo

The full set of [resources for the demo are here](https://github.com/kquinsland/the-missing-todoist-tools/blob/master/jobs/demo/), but the 'meaningful' parts are below:

```yaml

    # apply these labels...
    labels:
      - "at_work"

    # to tasks that...
    filters:
      - filter:
          # Any task that...
          task:
            # ... ends in 'at work' or 'the office'
            content:
              match: '(at work$|at the office$)'
              option:
                # remove portions of task content that match the query
                mutate: Yes
          # *AND*
          # ... has no labels
          labels:
            absent: Yes

          # *AND*
          # ... is in this project
          projects:
            name:
              match: 'Inbox'
```

and

```yaml

    # Create the projects...
    projects:
        # ... as child projects of `Garage Sale 🏚️💵`
      - parent:
          project: Garage Sale 🏚️💵

        # ... populated with template tasks/sections
        template:
          file: ./templates/garage-sale.csv

        # where the name of the projects to create comes from ...
        from:
          filters:
            - filter:
                # Any task that...
                task:
                  content:
                    match: "."
                    option:
                      delete: yes

                projects:
                  # ... is in this project
                  name:
                    match: 'Garage Sale 🏚️💵'

```

