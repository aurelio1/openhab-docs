---
layout: documentation
title: openHABian
source: https://github.com/openhab/openhabian/blob/master/docs/openhabian-DEBUG.md
---

<!-- Attention authors: Do not edit directly. Please add your changes to the appropriate source repository -->

This document is meant to give a guiding hand to users when their openHABian
install fails either on initial image installation or later on when running menu
options that install or configure optional components.

TL;DR:<br>
set `debugmode=maximum`in `/etc/openhabian.conf` and see `/boot/first-boot.log` for
image installation else record the terminal output.<br>
Do not ask for help on the forum unless you have FULLY read this guide.

**Attention:**
If you do not use the image but use `openhabian-config` manually - either to run
`openhabian-config unattended` or interactive use -, **there is no logfile**.
To record output in this case, you need to configure your terminal client to record
and save the command line output.
In PuTTy there's a field called 'Lines of scrollback' under the 'Window' option in
settings that you should increase to at least some thousand lines else you might not
capture everything you need to. Configure any other terminal client likewise.

Keep in mind that parts of the following information such as for example WiFi and
IPv6 setup don't apply to manually installed systems because they happen at or
before boot time.

## Prerequisites
First, please make sure you use the proper host hardware that is supported as
per [README](https://github.com/openhab/openhabian/blob/master/README.md).

openHABian requires a minimum of 1GB of RAM to run well. While you can get away
with a 512MB box like a RPi0W, you must not run anything other than openHAB
itself, in particular do **not** run memory hogs such as InfluxDB or Grafana.

openHABian requires you to provide direct Internet access. Using private IP
addresses is fine as long as your router properly provides NAT (Network Address
Translation) services.
Either Ethernet or WiFi is supported at install time, however, Ethernet tends to
be more reliable and WiFi requires user configuration prior to the first boot of
openHABian. To configure WiFi, simply edit the `wifi_psk=` and `wifi_ssid=`
fields in the `boot/openhabian.conf` file on your new SD card.

Your router (or a different device) needs to to provide properly configured DHCP
services so your openHABian box gets an IP address assigned when you boot it for
the first time.
The DHCP server also has to announce which DNS resolver to use so your box
knows how to translate DNS names into IP addresses.
It also needs to announce which IP address to use as the default gateway to the
internet - a typical access router is also the DHCP server will announce it's
own address here.
Finally, the DHCP server should also announce the NTP server(s) to use for
proper time services. Lack thereof will not break the installation procedure but
can lead to all sorts of long term issues so we recommend to setup DHCP to
announce a reachable and working NTP server.

A note on IPv6: openHABian was reported failing to boot in some environments
that make use of IPv6. If basic IP initialization fails (you cannot `ping`
your box) or installation gets stuck trying to download software packages, you
might want to try disabling IPv6. You can also do that before the very first
install attempt if you're sure you don't need any IPv6 connectivity on your
openHABian box. See [this section of openhabian.md](https://github.com/openhab/openhabian/blob/master/docs/openhabian.md#ipv6-notes)
how to disable IPv6 on your system.
Note that this is just a summary to cover the most commonly encountered cases.
The full boot procedure and how to obtain IP addresses, DNS resolver, default
route and NTP server addresses are highly complex and widely customizable and a
comprehensive description on how to properly configure your Internet access and
router are out of scope of openHABian. Please ask G\*\*gle how to accomplish that.

## Install
Proceed to installation: Etch-Burn-d(isk)d(ump)-Flash-whatever the image to an SD card.

NOW, read [openhabian.md](https://github.com/openhab/openhabian/blob/master/docs/openhabian.md#openhabianconf)
how to mount your SD card and how to modify the openHABian config file.
Some parameters are self-explanatory but please nonetheless read the full explanation
in the linked document.
Given that you're already reading the debug guide, the most important parameter
to set is likely `debugmode=maximum`.
Once you have passed the first time boot initialization phase and you can login
to the system, `/etc/openhabian.conf` will be used from there on. You can change
it at any time to get output on future boot runs or if you use `openhabian-config`
interactively.
_At this stage, read the first paragraph on the logfile and interactive use again._
To see debug output during the image installation process, you need to use the
procedure from your PC **before** you power your box on.

If you have a console available (monitor and keyboard), attach it to follow
the install process. Now insert the SD card and turn on your system.
If you don't have any console, try to access the web console at
`http://<yourhostip>:80/`.
It will display the contents of `/boot/first-log.boot` at intervals of some seconds
while installing.
Mind you that if installation fails, network access may or may not be possible
so you might need to access the box via console anyway in order to find out what
went wrong.

Login to your box via network using `ssh openhabian@<hostname>`.
The default hostname is `openhabiandevice`, and the default username and password
both are `openhabian` unless you changed either of these in `openhabian.conf`
before you started the installation run.
If that step fails, try to `ping openhabiandevice`. If that's failing, find out
the system's IP address (usually by looking at your router's running configuration
or using the command `arp -a` which is available on either Windows or Linux.
If you can login, there probably is an issue with your DHCP server.
If you cannot ping the system's IP address, try to login on the console.
In this case you will have to resort to debug the network setup which is beyond
the scope of this document. There's nothing specific about networking though -
openHABian just uses the setup of the underlying OS, Raspi OS for RPis that is or
whatever OS you chose to manually install upon. So again go ask G\*\*gle for help

Once logged in, enter `sudo bash` to become the root user.
Check if your install fails at some stage (also if it seems to hang forever):
there will exist a file either `/opt/openHABian-install-failed` or
`/opt/openHABian-install-inprogress` to reflect these states (to check,
`ls -l /opt/openHABian-install*`).
As a first troubleshooting step, you should reboot your box to see if the same
problems occurs on a second attempt.

## Debug
If the problem persists after booting succeeded at least in principle, login and
check `/boot/first-boot.log` to get an indication what went wrong in the install
process. You can avoid openHABian to start reinstalling on future reboots by
removing the status file, i.e. `rm -f /opt/openHABian-install*`, **but** be aware
that your installation is incomplete and that you should not run openHAB on a box
in that state.
You can use this state to debug, you can also use the menu options in
`openhabian-config` to manually install everything that failed or is missing.
See `/opt/openhabian/openhabian-setup.sh` and the corresponding code in
`/opt/openhabian/functions/*.bash` what usually gets installed on an unattended
installation. Note that if you keep or recreate the status file (just `touch
/opt/openhabian-install-failed`), you can reboot at any time to continue
unattended installation. So if say Java install fails (Java being a prerequisite
to installation of openHAB), you can use `openhabian-config` or manual install,
then continue installation by rebooting.
Should you succeed at some point in time - great! Let us know what you did to
make it work please through a Github issue (see below).
As we cannot be sure everything on your box is 100% the same what an
unattended install gets you, please also do a complete reinstall before you
start operating openHAB. If possible start with the flash step. If that does
not work, at least delete all the packages that openhabian-setup had installed
before you reboot. Use `apt purge` (and not just `apt remove`).

### Create a debug log
You can put openHABian into a more verbose debug level **at any time** after
the very first installation run: edit the config file `/etc/openhabian.conf`
using the editor of your choice (use `nano` if you have no idea) and change
the `debugmode` parameter to either `on` or `maximum` right away (default
is `off`). If it's missing there, just add it.
Specifying `maximum` is usually your best choice as it will have
`openhabian-config` show every single command it executes so you might spot
the problem right away. If you open an issue, always provide the maintainers
with a logfile at `maximum` detail level.

Your next boot run will also exhibit much more verbose logging. Remember boot
time output will be appended to `/boot/first-boot.log`.
If installation still fails to finish, please retrieve that file from your box,
open a GitHub issue (see next paragraph), thoroughly describe the environment
conditions and your findings so far and upload the log.

### How to open a Github issue
While written for openHAB, the guideline at <https://community.openhab.org/t/how-to-file-an-issue/68464>
also applies to openHABian issues.
Please proceed as told there. openHABian has its own repository at <https://github.com/openhab/openhabian>.
Search the issues listed there first if 'your' problem has already been seen and
eventually opened as an issue by someone else (you should remove the `is:open` filter from the search
bar to let you see closed issues). If so, you may leave a "me too"
comment there but please do not open another issue to avoid duplicates.
You can reference other issues (eventually also request to reopen closed ones)
and Pull Requests by their number (just type #XXX along with your text,
GitHub will insert the proper link).
If you open an issue, we kindly ask you to deliver as much information as
possible. It is awkward and annoying if we need to spend time asking and asking
what the real problem is about. Please avoid that situation, be proactive and
tell us in the first place.
Once you opened the issue, copy `/boot/first-boot.log` from your openHABian box
over to your desktop and upload it to GitHub.
If you succeed logging on and get to see a banner with system information,
please also copy that as part of your issue.

If you're able to help in producing a fix to problems, we happily take any
Pull Request.
Explaining git and Github unfortunately is out of our scope (Google is your
friend).
See the guidelines outlined in [CONTRIBUTING.md](https://github.com/openhab/openhabian/blob/master/CONTRIBUTING.md)
as well.
For simple fixes to a single file only, you can click through the source
starting at <https://github.com/openhab/openhabian> and edit the file online,
GitHub will then offer to create the PR.
You can also clone the openHABian repository, make your changes locally and use
git to check in your changes and upload them to a repo copy of yours, then
follow the git-offered link to create the PR.

## Checkpoint
Remember to always let `openhabian-config` update itself on start.

If you want to change anything to work around some not yet fixed bug, you can
directly edit the files in and below `/opt/openhabian` on your box. Just do not
let `openhabian-config` update itself on start as that would overwrite your
changes.
You can also clone (download) a different openHABian version than the most
current one, e.g. if a maintainer or contributor to openHABian offers or asks
you to test-drive a development version. Set the `clonebranch` parameter in
`/etc/openhabian.conf` to the branch name to load, then update `openhabian-config`
on start.
**Note**: You must not modify `repositoryurl` to point elsewhere than the
official repo. openHABian will only ever update from there so you can only
test drive a test branch that a developer has provided you on the official site.

The main program is in `openhabian-setup.sh`.
If the initial unattended install fails again and again at the same step (say
Java installation), you may try to comment that step out. But mind the code in
`build-image/first-boot.bash` towards the end starting with `git clone`.
This is where openHABian updates itself. If you don't comment that out as well,
it'll overwrite your changes on the next install run.

## Disclaimer
For obvious reasons, changing openHABian code is not a supported procedure.
We just want to give you a hint what you _could_ try doing if your install fails
and you're sitting there, desperately looking for a fix.
Google and learn for yourself how to edit a file, learn to understand shell
programming basics, you're on your own here.
If you change openHABian code on your box, remember for the time it takes to get
openHABian officially fixed, you must not let `openhabian-config` update itself
on start as that would overwrite your changes.