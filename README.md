# smokeping-mtr-alert

When SmokePing detects an [alert](http://oss.oetiker.ch/smokeping/doc/smokeping_config.en.html#___top) condition it can be configured to run a script instead of sending an e-mail.

This script, when launched from SmokePing, runs an [MTR](http://www.bitwizard.nl/mtr/) traceroute in report mode and e-mails the output to the designated address. This can be useful in providing some clues as to the source of the packet loss.

## Installation

This script requires that MTR and sendmail are installed and available on the path of the user running SmokePing.

`smokeping-mtr-alert` should be placed in a directory that is readable by the SmokePing user (such as `/usr/local/bin`), and marked as executable. As SmokePing does not permit arguments to be passed to alert scripts you may wish to use a wrapper script to override configuration (such as the destination e-mail address or system name). An example wrapper script is provided in `smokeping-mtr-alert-wrapper`.

SmokePing may then be configured in its "pipe" mode to run the script when an alert is triggered. The "from" address can be anything as it is not passed to the script.

<pre>
*** Alerts ***
to = |/usr/local/bin/smokeping-mtr-alert
from = smokeping@example.com

+someloss
type = loss
# in percent
pattern = >0%,*12*,>0%,*12*,>0%
comment = loss 3 times  in a row
</pre>

Configured alerts may then be attached to targets.

<pre>
++ AlertDemo

menu = Demonstration
title = Demonstration host for alerts
host = 192.0.2.1
alerts = someless
</pre>

More information about the alerting syntax is available [in the SmokePing documentation](http://oss.oetiker.ch/smokeping/doc/smokeping_config.en.html#___top).

## Example e-mail

<pre>
From: smokeping-mtr-alert@monitoring.example.com
To: root@example.com
Subject: monitoring SmokePing alert: Test.AlertDemo
Date: Thu,  2 Jun 2016 13:27:22 +1200 (NZST)

Packet loss report from monitoring for Test.AlertDemo at Thu Jun  2 13:27:22 2016.

mtr -n --report 192.0.2.1

Start: Thu Jun  2 13:27:12 2016
HOST: monitoring                 Loss%   Snt   Last   Avg  Best  Wrst StDev

Alert triggered: anyloss
Target: Test.AlertDemo
Target hostname: 192.0.2.1
Loss pattern: 100%
RTT: U
</pre>

## Usage

The positional arguments to the script will usually be provided by SmokePing itself.

<pre>
usage: smokeping-mtr-alert [-h] [--email EMAIL] [--name NAME]
                           alert target loss_pattern rtt hostname

Run from SmokePing as a "pipe" alert target. Sends an MTR for the target to
the designated e-mail address. Michael Fincham
&lt;michael.fincham@catalyst.net.nz&gt;.

positional arguments:
  alert          name of the alert, supplied by smokeping
  target         target being monitored, supplied by smokeping
  loss_pattern   loss pattern that has triggered, supplied by smokeping
  rtt            current RTT for target, supplied by smokeping
  hostname       hostname of target, supplied by smokeping

optional arguments:
  -h, --help     show this help message and exit
  --email EMAIL  e-mail address to send report, defaults to root
  --name NAME    name of smokeping installation, defaults to hostname where
                 the script runs
</pre>

## Limitations

* No support for the `edgetrigger` option in SmokePing.
* SmokePing will not run a script with custom parameters, necessitating the use of a wrapper script if defaults need to be overridden.
