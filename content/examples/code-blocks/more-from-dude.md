+++
title = "View GitHub Pull Requests in Magit"
author = ["Kaushal Modi"]
description = """
  How to view GitHub Pull Request branches locally in the cloned repo,
  and more importantly, how to do that automatically from within Emacs.
  """
date = 2022-06-23T17:51:00-04:00
tags = ["git", "magit", "100DaysToOffload", "github", "git-reference"]
categories = ["emacs", "elisp"]
draft = false
creator = "Emacs 28.1.91 (Org mode 9.5.4 + ox-hugo)"
[versions]
  emacs = "28.1.50"
[syndication]
  mastodon = 108529062204527384
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Locally creating a branch for a PR](#locally-creating-a-branch-for-a-pr)
- [Getting references to all Pull Requests](#getting-references-to-all-pull-requests)
- [Automatically adding PR refs](#automatically-adding-pr-refs)
- [Summary](#summary)
- [References](#references)

</div>
<!--endtoc-->


I have a few public projects in git repos, but I don't get that much
traffic in _Pull Requests (PR)_
{{% sidenote %}}
Gitlab calls these _Merge Requests_ or MRs.
{{% /sidenote %}} . So when I need to add additional commits to a PR, I would just add
the PR author's _remote_ to my local repo, _check out_ their PR
branch, add my own commits and then merge that to my project's _main_
branch.

As these occurrences were few and far apart, I didn't have a need to
view the _PR branches_ directly from within Emacs/Magit. Though, I
somehow knew that each GitHub Pull Request's _HEAD_ got assigned a [git
**reference**](https://git-scm.com/book/en/v2/Git-Internals-Git-References). But I didn't need to use that knowledge until today
:smiley:.

Today, when discussing [PR # 20](https://github.com/protesilaos/denote/pull/20) on [Prot's](https://protesilaos.com/) [Denote](https://protesilaos.com/emacs/denote) package's GitHub
mirror
{{% sidenote %}}
Prot uses [SourceHut](https://git.sr.ht/~protesilaos/denote) as the primary git forge for his Emacs
packages. But I am glad that he doesn't mind the activity in Issues
and Pull Requests on the GitHub mirror.
{{% /sidenote %}} , he wrote [this comment](https://github.com/protesilaos/denote/pull/20#issuecomment-1164676013):

> Now I just need to figure out how best to incorporate your changes
> into the `org-id` branch so I can add the final bits. I am not too
> familiar with the PR workflow ...

.. and that inspired this post today.


## Locally creating a branch for a PR {#locally-creating-a-branch-for-a-pr}

From the [GitHub docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/checking-out-pull-requests-locally), the `git` command to create a local branch for a
PR is this:

```shell
git fetch origin pull/ID/head:BRANCHNAME
```

I have the `denote` package cloned from its GitHub mirror. So the
**origin** remote's **url** is `https://github.com/protesilaos/denote`.

<div class="note">

Make sure that the remote name used in this command is pointing to a
GitHub repo, and not a mirror forge like GitLab or SourceHut.

</div>

When I ran the below command, I got a new branch **pr-20** pointing to
the latest commit of that PR:

```shell
git fetch origin pull/20/head:pr-20
```

Awesome!

<div class="verse">

.. But that wasn't good enough<br />
&nbsp;&nbsp;&nbsp;&nbsp;.. Now I wanted more<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.. I didn't want to manually create a branch for each PR.<br />

</div>


## Getting references to all Pull Requests {#getting-references-to-all-pull-requests}

Now that I was on that quest of "I want more", it didn't take me long
to re-discover [this 7-year old nugget](https://oremacs.com/2015/03/11/git-tricks/#illusion-2-quickly-get-github-pull-requests-on-your-system) by [Oleh Krehel](https://oremacs.com). Here are the
relevant bits from that post:

1.  Open the local repo's `.git/config` file.
2.  Find the `[remote "origin"]` section
3.  Modify it by adding this one line with **pull** refs. _This is the
    same for all GitHub repositories._
    ```cfg { hl_lines=["4"] }
    [remote "origin"]
        url = https://github.com/USER/REPO.git
        fetch = +refs/heads/*:refs/remotes/origin/*
        fetch = +refs/pull/*/head:refs/pull/origin/*
    ```

With that edit in place, when I did <kbd>l</kbd>&nbsp;<kbd>a</kbd> (show
the log for all git references), followed by <kbd>f</kbd>&nbsp;<kbd>a</kbd>
(fetch all the remotes) in the Magit, I could see the references to
the `denote` repo's PRs!

<a id="figure--magit-denote-pr-refs"></a>

{{< figure src="pr-20-ref.png" caption="<span class=\"figure-number\">Figure 1: </span>Viewing PR references from `denote` package's GitHub repo" >}}

<div class="verse">

.. But that still wasn't good enough<br />
&nbsp;&nbsp;&nbsp;&nbsp;.. I didn't want to manually edit the `.git/config` in each repo.<br />

</div>


## Automatically adding PR refs {#automatically-adding-pr-refs}

Of course, I wasn't the first one to think of this!

Another Emacs veteran [Artur Malabarba](https://endlessparentheses.com/) had already had this covered
also around [7 years back](https://endlessparentheses.com/automatically-configure-magit-to-access-github-prs.html). Coincidentally, that post was written as a
response to that same blog post by Oleh where he shared the above
`.git/config` tip.

In that post, Artur shares an Emacs Lisp function that uses Magit
functions like `magit-get`, `magit-get-all` and `magit-git-string` to
auto-add the `fetch = +refs/pull/*/head:refs/pull/origin/*` line in
the `.git/config`. This magic happens after checking that the **origin**
remote points to a GitHub repo, and if that line doesn't already
exist.

Here, I am lightly modifying the function shared in that post so that
the **origin** remote name is not hard-coded
{{% sidenote %}}
The reason is that sometimes, I name the original remote as **upstream**
and my fork as **fork**, and I might have no remote named **origin**.
{{% /sidenote %}} . Credit for the main logic in this code still goes to Artur.

<a id="code-snippet--add-PR-fetch-ref"></a>
```emacs-lisp
(defun modi/add-PR-fetch-ref (&optional remote-name)
  "If refs/pull is not defined on a GH repo, define it.

If REMOTE-NAME is not specified, it defaults to the `remote' set
for the \"main\" or \"master\" branch."
  (let* ((remote-name (or remote-name
                          (magit-get "branch" "main" "remote")
                          (magit-get "branch" "master" "remote")))
         (remote-url (magit-get "remote" remote-name "url"))
         (fetch-refs (and (stringp remote-url)
                          (string-match "github" remote-url)
                          (magit-get-all "remote" remote-name "fetch")))
         ;; https://oremacs.com/2015/03/11/git-tricks/
         (fetch-address (format "+refs/pull/*/head:refs/pull/%s/*" remote-name)))
    (when fetch-refs
      (unless (member fetch-address fetch-refs)
        (magit-git-string "config"
                          "--add"
                          (format "remote.%s.fetch" remote-name)
                          fetch-address)))))
(add-hook 'magit-mode-hook #'modi/add-PR-fetch-ref)
```
<div class="src-block-caption">
  <span class="src-block-number"><a href="#code-snippet--add-PR-fetch-ref">Code Snippet 1</a>:</span>
  Function to auto-add GitHub PR references to the repo's <code>.git/config</code>
</div>


## Summary {#summary}

With the above snippet added to your Emacs config and evaluated, each
time you visit a repo cloned from GitHub in the Magit Status buffer
(`M-x magit-status`), the PR refs will get auto-added to that repo's
`.git/config` if needed.

After that, you can easily view the commits from all the PRs by doing
<kbd>l</kbd>&nbsp;<kbd>a</kbd>&nbsp;<kbd>f</kbd>&nbsp;<kbd>a</kbd>.


## References {#references}

-   [oremacs -- Some git/magit/github tricks](https://oremacs.com/2015/03/11/git-tricks/#illusion-2-quickly-get-github-pull-requests-on-your-system)
-   [endlessparentheses -- Automatically configure Magit to access Github PRs](https://endlessparentheses.com/automatically-configure-magit-to-access-github-prs.html)
-   [GitHub -- Checking out PR branches locally](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/checking-out-pull-requests-locally)

[//]: # "Exported with love from a post written in Org mode"
[//]: # "- https://github.com/kaushalmodi/ox-hugo"
