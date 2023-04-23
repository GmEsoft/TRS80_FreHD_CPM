Procedure to build a FreHD CP/M bootable system on TRS-80 Model 4/4p
====================================================================

SYSGENing Montezuma Micro CP/M 2.2 BIOS 2.22+ to a hard disk
------------------------------------------------------------

### Needed

- a TRS-80 Model 4/4p;
- a 15 MB Radio-Shack Hard Disk or a FrehD emulator;
- a bootable Montezuma Micro CP/M 2.2 floppy, with bios 2.22 or 2.3x;
- a disk containing the Hard Disk support files for CP/M (MMHARD);
- a disk containing the FreHD tools (VHDUTL.COM, IMPORT2.COM, EXPORT2.COM).


### Steps

1.	Boot with CP/M system disk in A:

2.	Format a blank disk mounted in B:
	```
	A>DUP
	```
	Select:
	- `A` - Format a disk
	- `B` - Drive B:
	
	`ENTER` to start formatting (ENTER to confirm)
	
	`^C` three times when complete
	
3.	Create a 63K CP/M system
	```
	A>MOVCPM 63 *
	```

4.	Save the new prepared system to disk using SYSGEN
	```
	A>SYSGEN
	SOURCE DRIVE:			<ENTER>
	DESTINATION DRIVE:		B<ENTER>
	INSERT DISK IN B:		<ENTER>
	DESTINATION DRIVE:		<ENTER>
	```

5.	Copy CP/M files from A: to B:
	```
	A>PIP B:=A:PIP.COM
	A>PIP B:=A:SUBMIT.COM
	A>PIP B:=A:KEYDEF.COM
	A>PIP B:=A:SYSGEN.COM
	A>PIP B:=A:EXBIOS.COM
	A>PIP B:=A:CONFIG.COM
	```

6.	Mount the new disk in B: into drive A:
	- Press RESET (on 4p, press 2/F2 to force booting on floppy)

7.	Mount the disk containing the FreHD CP/M Utilities in B:

8.	Copy the utilities to A:
	```
	A>PIP A:=B:VHDUTL.COM
	```

9.	Mount the disk containing the MM Hard Disk Drivers in B:

10.	Copy the following files to A:
	```
	A>PIP A:=B:BUILD.COM
	A>PIP A:=B:CPMFIX.COM
	A>PIP A:=B:HDRS15M.COM
	```

11.	VHDUTL for CP/M doesn't work when booted from floppy.
	```
	Message: Interface not found.
	```
	Cause: ENEXTIO (MODOUT.4) is not active in CP/M if booted from floppy.
	
	Boot a DOS system and run VHDUTL from there. Or get a fixed version
	of VHDUTL.

12.	Create a new virtual hard drive:
	```
	VHDUTL (MNT,ADDR=1,VHD= "filename",CREATE,HALT)
	```
	Creates a new virtual hard disk file:
	- ADDR=1: drive address (1=primary, 2=secondary)
	- CREATE will create an ST-251 equivalent:
		- Cylinders	840
		- Heads		6
		- Sectors/Cyl	192
	```
	=> Mounted OK.  Press Reset
	```

13.	Mount the new CP/M disk into drive A:
	- Press RESET (on 4p, press 2/F2 to force booting on floppy)

14.	Format the new hard disk
	```
	A>EXBIOS                        BIOS extension for DS disks
	A>CPMFIX                        Displays the user number
	A0>HDRS15M F=ABCD H0=EF INIT	1 15MB disk with 2 partitions on each
	                                E: and F: on 1st HD
	                                Floppies are assigned to A: B: C: D:
	```
	- No flawed track to specify.
	- This creates 2 partitions of 7.5MB each.
	- Verifying takes much more time than formatting... total 20 mins
	- Don't init H1 if no image is mounted!

15.	Press RESET (on 4p, press 2/F2 to force booting on floppy)

16.	Build the HDBOOT.SUB file:
	```
	A0>BUILD HDBOOT
	EXBIOS
	CPMFIX
	HDRS15M H0=AB H1=CD F=EFGH
	(blank line to terminate)
	A0>
	```

	- 2 15MB disks with 2 partitions on each.
	  A: and B: on 1st HD, C: and D: on 2nd
	- Floppies are assigned to E: F: G: H:.

17.	Submit the HDBOOT.SUB FILE:
	```
	A>SUBMIT HDBOOT
	```

18.	Press RESET (on 4p, don't press 2/F2 to boot from HD)
	- The system should now boot on the hard disk on the
	model 4p (without the modified boot ROM). With modified 
	boot ROMs it may not succeed, as the 'bootable' flag
	was not set while creating the disk image. See step 22.

19.	Mount the CP/M System Disk in drive E: (floppy A:)
	- Copy all files from it to the new hard disk.
	```
	A0>E:PIP A:=E:*.*
	```
20.	Mount the disk containing the FreHD CP/M Utilities in E:
	- Copy all files from it to the new hard disk.
	```
	A0>PIP A:=E:*.*
	```
21.	Mount the disk containing the MM Hard Disk Drivers in E:
	Copy all files from it to the new hard disk.
	```
	A0>PIP A:=E:*.*
	```
22.	The VHD file must be patched in order to be recognized by
	the FreHD Boot Selector.
	- Sector 0 byte 0x08 = 0x01 (Bootable flag)
	- Sector 0 byte 0x0B = 0x02 (Type: 2=4p natively bootable OS)


### Notes

- This procedure assumes a 6-head and 840-cylinder 15 MB hard drive image. 
  This hard drive is compatible with the driver HDRS15M. This driver allows 
  the following logical drive layouts (heads and sizes):

  | # logical drives |  Drive 1   |  Drive 2   |  Drive 3   |  Drive 4   |
  |------------------|----------- |------------|------------|------------|
  |          1       | 0-5:  15MB |     --     |     --     |     --     |
  |          2       | 0-2: 7.5MB | 3-5: 7.5MB |     --     |     --     |
  |          3       | 0,1:   5MB | 2,3:   5MB | 4,5:   5MB |     --     |
  |          4       | 0-2: 7.5MB |   3: 2.5MB |   4: 2.5MB |   5: 2.5MB |
  
  The layout presented in this procedure is the second layout in the table.
  
  Note that the maximum size for a CP/M partition is 8 MB. So, by selecting the
  first layout with a unique partition, 7 MB of space would be wasted.
  
  The other layouts can also be chosen. The commands to use for both the 
  initialization and the HDBOOT.SUB files are:
  
  | # logical drives |        Initialization         |        HDBOOT.SUB        |
  |------------------|-------------------------------|--------------------------|
  |          1       | `HDRS15M F=ABCD H0=E INIT`    | `HDRS15M H0=A F=EFGH`    |
  |          2       | `HDRS15M F=ABCD H0=EF INIT`   | `HDRS15M H0=AB F=EFGH`   |
  |          3       | `HDRS15M F=ABCD H0=EFG INIT`  | `HDRS15M H0=ABC F=EFGH`  |
  |          4       | `HDRS15M F=ABCD H0=EFGH INIT` | `HDRS15M H0=ABCD F=EFGH` |

- If the VHD filename is not specified to VHDUTL, the default name `HARD4-0` will
  be used. This filename allows the Model 4p with unmodified ROM to boot natively
  to the hard disk image. It is not necessary in that case to patch the VHD file
  as described in step 22. The patch is necessary only for the FreHD Boot Selector.
  
  
### Tested on

- MM CP/M vers 2.2 BIOS vers 2.22 US	: OK
- MM CP/M vers 2.2 BIOS vers 2.30 US	: OK
- MM CP/M vers 2.2 BIOS vers 2.31 US	: OK
- MM CP/M vers 2.2 BIOS vers 2.31 FR	: OK
