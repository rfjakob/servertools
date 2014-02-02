servertools
===========

Scripts to make server administration (especially backups) easier.

Included tools (more info may be found in the header of each file):

rsync-mirror-wrapper
--------------------
Mirror a directory using rsync, setting all the right command line
options, writing a log file and handling "file has vanished" errors.

Usage example:

	cd /mirrors/foo.com
	rsync-mirror-wrapper foo.com:/ rootfs

When finished, you get a directory structure like this:

	rootfs/
	rootfs.rsync.log
	rootfs.rsync-mirror-started
	rootfs.rsync-mirror-success

mysql-consisnap
---------------
Take a fully consistent (including MyISAM tables!) LVM snapshot of a
MySQL DB.

It tries hard to never leave the database locked for longer than
neccessary. The maximum lock time is 60 seconds - if anything takes
longer than that the lock will be released and an error message
printed.

Needs Perl DBI: apt-get install libdbi-perl libdbd-mysql-perl

Modus operandi:

1) Connect to MySQL server, flush out dirty data by executing
   FLUSH TABLES;
2) Lock all databases to get them into a consistent state by executing
   FLUSH TABLES WITH READ LOCK;
3) Create LVM snapshot
4) Disconnect from MySQL and release the lock
5) Mount snapshot

Idea from http://www.butschek.de/fachartikel/perl-mysql-backup/

Usage example:

	# Take snapshot of database partition
	ssh foo.com mysql-consisnap /etc/mysql-consisnap.conf
	# And make sure it is dropped again when we finish or are killed
	trap "ssh foo.com mysql-consisnap /etc/mysql-consisnap.conf -d" INT TERM EXIT
	# Rsync the snapshot
	cd /mirrors/foo.com
	rsync-mirror-wrapper foo.com:/mnt/mysql-consisnap/ foo-mysql-rootfs
