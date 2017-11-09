# Git authentication tools for Google Compute Engine

The `git-cookie-authdaemon` uses the GCE metadata server to acquire an
OAuth2 access token and configures `git` to always present this OAuth2
token when connecting to googlesource.com or
[Google Cloud Source Repositories][CSR].

[CSR]: https://cloud.google.com/source-repositories/

## Setup

Launch the GCE VMs with the gerritcodereview scope requested, for example:

```
gcloud compute instances create \
  --scopes https://www.googleapis.com/auth/gerritcodereview \
  ...
```

To add a scope to an existing GCE instance see this
[gcloud beta feature](https://cloud.google.com/sdk/gcloud/reference/beta/compute/instances/set-scopes).

## Installation on Linux

Install the daemon within the VM image and start it running:

```
sudo apt-get install git
git clone https://gerrit.googlesource.com/gcompute-tools/
./gcompute-tools/git-cookie-authdaemon
```

The daemon launches itself into the background and continues
to keep the OAuth2 access token fresh.

## Installation on Windows

1. Install [Python](https://www.python.org/downloads/windows/) and [Git](https://git-scm.com/download) for Windows if they are not.
1. Run `git-cookie-authdaemon` in the same environment under the same user
   git commands will be run, for exmaple in either Command Prompt or Cygwin bash
   shell for user `builder`.
```
python git-cookie-authdaemon --nofork
```

### Lauch at Windows boot

It may be desired in automation to launch `git-cookie-authdaemon` at
Windows boot. It can be done as a scehduled task. The following is an
example on a Jenkins node. The setup is:

1. The VM is created from GCE Windows Server 2012R2 image.
1. Gygwin with SSHD is installed.
1. Jenkins master launches the node through SSH as `builder` account.

How to create a scheduled task.

1. Launch Task Scheduler from an Administrator account.
1. In `General` tab:
   1. Change user to the one running Jenkins node if it is different. You may
      want to run Jenkins node as a non-priviledged user, `builder` in this
      example.
   1. Select `Run whether user is logged on or not`
1. In `Trigger` tab. Add a trigger
   1. `Begin the task` as `At startup`.
   1. Uncheck `Stop task if it runs longer than`.
   1. Check `Enabled`.
1. In `Actions` tab.  Add `Start a program`.
   1. `Program/script` as `C:\cygwin64\bin\bash.ext`,
   2. `Add arguments` as
      `--login -c /home/builder/git-cookie-authdaemon_wrapper.sh` (see note
      below)

Note: debugging a scheduled task is not streightforward. A wrapper like
`/home/builder/git-cookie-authdaemon_wrapper.sh` below can be helpful to capture git-cookie-autodaemon.log errors.

```
python <path_to>/git-cookie-authdaemon --nofork >> git-cookie-autodaemon.log  # option --debug is also available.
```
