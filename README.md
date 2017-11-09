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

1. Install [Python](https://www.python.org/downloads/windows/) and
   [Git](https://git-scm.com/download) for Windows.
1. Run `git-cookie-authdaemon` in the same environment under the same user
   git commands will be run, for example in either `Command Prompt`
   or `Cygwin bash shell` under user `builder`.
```
python git-cookie-authdaemon --nofork
```

### Launch at Windows boot

It may be desired in automation to launch `git-cookie-authdaemon` at
Windows boot. It can be done as a scheduled task. The following is an
example on a Jenkins node. The setup is:

1. The VM is created from GCE Windows Server 2012R2 image.
1. Gygwin with SSHD is installed.
1. The Jenkins master connects to the VM through SSH as `builder` account.

How to create a scheduled task.

1. Launch `Task Scheduler` from an Administrator account.
1. Click `Create Task` in the right pane.
1. In `General` tab:
   1. Change user to the one running Jenkins node if it is different. You may
      want to run Jenkins node as a non-privileged user, `builder` in this
      example.
   1. Select `Run whether user is logged on or not`
1. In `Trigger` tab. Add a trigger
   1. Set `Begin the task` as `At startup`.
   1. Uncheck `Stop task if it runs longer than`.
   1. Check `Enabled`.
1. In `Actions` tab.  Add `Start a program`.
   1. Set `Program/script` as `C:\cygwin64\bin\bash.ext`,
   1. Set `Add arguments` as
      `--login -c /home/builder/git-cookie-authdaemon_wrapper.sh` (see note
      below)
1. Click `Ok` to save it.
1. Optional: click `Enable All Tasks History` in `Task Scheduler`'s right pane.
1. Add `builder` account to `Administrative Tools -> Local Security Policy ->
   Local Policies -> User Rights Assignment -> Log On As Batch Job`

Note: /home/builder/git-cookie-authdaemon_wrapper.sh` below does

1. Set HOMEPATH if it is not.
2. Capture git-cookie-autodaemon.log stdout and stderr for debugging.

```
#!/bin/bash
exe=gcompute-tools/git-cookie-authdaemon
log=/cygdrive/c/build/git-cookie-autodaemon.log

# HOMEPATH is not set in task scheduled at machine boot.
export HOMEPATH=${HOMEPATH:-'\Users\builder'}

/cygdrive/c/Python27/python $exe --nofork >> $log 2>&1 # option --debug is also available.
```
