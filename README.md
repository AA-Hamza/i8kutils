# i8kutils with cpu temp averaging

* Use i8kmon for simple average (t1+t2+t3+t4)/4
* Use i8kmon2 (Which I personally use) for realtime averageing (curr+prev)/2

# Package up for adoption

https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=948521

https://www.debian.org/devel/wnpp/rfa






I8KUTILS
========

OVERVIEW
========

i8kutils package contains user-space programs for controlling the fans on some Dell
laptops.

Note: i8kutils is entirely built upon the dell-smm-hwmon kernel module.

These data contains the states and the system temperature along
with others infos. The fields returned in a query to the system are
summarized below.

    * BIOS version

    * Dell service tag (later known as 'serial number')

    * CPU temperature

    * fan status

    * fan rotation speed (only on some models)

    * ac power status

    * volume buttons status (not the multimedia buttons)

The data are collected from the dell-smm-hwmon kernel module that is included in recent
kernels.

The i8kutils package includes the following utilities:

    * i8kctl	 - command-line interface to the kernel module

    * i8kmon   - temperature monitor with fan control capability

    * i8kfan   - utility to set state (speed) of fans

The i8kctl perform queries and sets related to fan control as
read temperature, turn the fan on. The i8kmon continuously monitor the
system temperature and control automatically the fans.

All Dell laptop has the feature of controlling the temperature in the BIOS, but
to some models this feature does not work properly. i8kmon does essentially the same
job as the BIOS is supposed to do.

The latest version of the i8kutils package can be retrieved at:

    https://launchpad.net/i8kutils

The module is supposed to be loaded in the system to i8kmon service starts.


LICENCE
=======

This software is released under the terms of the GNU General Public
Licence.

   Copyright (C) 2001-2009 Massimo Dal Zotto <dz@debian.org>

   This program is free software; you can redistribute it and/or modify
   it under the terms of the GNU General Public License as published by
   the Free Software Foundation; either version 2 of the License, or
   (at your option) any later version.

   This program is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
   GNU General Public License for more details.

   You should have received a copy of the GNU General Public License
   along with this program; if not, write to the Free Software Foundation,
   Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301, USA.

On Debian GNU/Linux systems, the complete text of the GNU General
Public License can be found in `/usr/share/common-licenses/GPL'.


THE KERNEL DRIVER
=================

The information provided by the kernel driver can be accessed by simply
reading the /proc/i8k file. For example:

    $ cat /proc/i8k
    1.0 A17 2J59L02 52 2 1 8040 6420 1 2

The fields read from /proc/i8k are:

    1.0 A17 2J59L02 52 2 1 8040 6420 1 2
    |   |   |       |  | | |    |    | |
    |   |   |       |  | | |    |    | +------- 10. buttons status
    |   |   |       |  | | |    |    +--------- 9.  ac status
    |   |   |       |  | | |    +-------------- 8.  right fan rpm
    |   |   |       |  | | +------------------- 7.  left fan rpm
    |   |   |       |  | +--------------------- 6.  right fan status
    |   |   |       |  +----------------------- 5.  left fan status
    |   |   |       +-------------------------- 4.  CPU temperature (Celsius)
    |   |   +---------------------------------- 3.  Dell service tag (later known as 'serial number')
    |   +-------------------------------------- 2.  BIOS version
    +------------------------------------------ 1.  /proc/i8k format version

A negative value, for example -22, indicates that the BIOS doesn't return
the corresponding information. This is normal on some models/bioses.

For performance reasons the /proc/i8k doesn't report by default the ac status
since this SMM call takes a long time to execute and is not really needed.
If you want to see the ac status in /proc/i8k you must explictitly enable
this option by passing the "power_status=1" parameter to insmod. If ac status
is not available -1 is printed instead.

The driver provides also an ioctl interface which can be used to obtain the
same information and to control the fan status. The ioctl interface can be
accessed from C programs or from shell using the i8kctl utility. See the
source file i8kctl.c for more information on how to use the ioctl interface.

The driver accepts the following parameters:

    force=1

	force loading of the driver on unknown hardware.

    restricted=1

	allow fan control only to processes with the CAP_SYS_ADMIN capability
	set or processes run as root. In this case normal users will be able
	to read temperature and fan status but not to control the fan.
	If your notebook is shared with other users and you don't trust them
	you may want to use this option.

    power_status=1

	report ac status in /proc/i8k. Default is 0.

    repeat_delay=<delay>

	specifies the delay before the driver will start generating repeat
	events when a button is kept pressed. Default is 250ms.
	This option is available only with kernel 2.4.

    repeat_rate=<rate>

	specifies the button repeat rate. Default is 10 times for second.
	This option is available only with kernel 2.4.

You can specify the module parameters when loading the module or as kernel
option when booting the kernel if the driver is compiled statically.

To have the module loaded automatically at boot you must manually add the
line "dell-smm-hwmon" into the file /etc/modules or use the modconf utility. For example:

    $ cat /etc/modules
    # /etc/modules: kernel modules to load at boot time.
    #
    # This file contains the names of kernel modules that should be loaded
    # at boot time, one per line. Lines beginning with "#" are ignored.
    dell-smm-hwmon

Any module parameters must be specified in /etc/modprobe.d/dell-smm-hwmon.conf.
Recent kernels required the option below to work.

    $ cat /etc/modprobe.d/dell-smm-hwmon.conf 
    options dell-smm-hwmon restricted=0


THE I8KCTL UTILITY
==================

The i8kctl utility provides a command-line interface to the dell-smm-hwmon kernel driver.
When invoked without arguments the program reports the same information which
can be read from the /proc/i8k file.

A main difference between 'i8kctl' and '/proc/i8k' is that i8kctl gets and sets
parameters in real-time. What not happens in /proc/i8k which is updated from
time to time by the kernel module.

The program can take an optional argument which can be used to select only one
of the items and to control the fan status.


COMPILATION
===========

To compile the programs type "make".


CONTRIBUTORS
============

Contributors are listed here, in alphabetical order.

    Pablo Bianucci <pbian@physics.utexas.edu>

	support for /proc/acpi

    David Bustos <bustos@caltech.edu>

	patches for generating keyboard events

    Jonathan Buzzard <jonathan@buzzard.org.uk>

	basic information on the SMM BIOS and the Toshiba SMM driver.
	Asm code for calling the SMM BIOS on the I8K. Without his help
	this work wouldn't have been possible.

    Karl E. Jørgensen <karl@jorgensen.com>

	init script for i8kmon daemon

    Stephane Jourdois <stephane@tuxfinder.org>

	patches for correctly interpreting buttons status in the i8k driver

    Marcel J.E. Mol <marcel@mesa.nl>

	patches for the --repeat option in the i8kbuttons (obsolete on Abr 30, 2014) util

    Gianni Tedesco <gianni@ecsc.co.uk>

	patch to restrict fan contol to SYS_ADMIN capability

    David Woodhouse <dwmw2@redhat.com>

  suggestions on how to avoid the zombies in i8kbuttons (obsolete on Abr 30, 2014)

    Vitor Augusto <vitorafsr@gmail.com>

  fixes for the freeze bug at i8kmon, general update and bug fixes

and many others who tested the driver on their hardware and sent reports
and patches.

No credits to DELL Computer who has always refused to give support on Linux
or provide any useful information on the I8K buttons and their buggy BIOS.


--
Massimo Dal Zotto <dz@debian.org>
