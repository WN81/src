<!--
 - Copyright (C) 2017, 2018  Internet Systems Consortium, Inc. ("ISC")
 -
 - Permission to use, copy, modify, and/or distribute this software for any
 - purpose with or without fee is hereby granted, provided that the above
 - copyright notice and this permission notice appear in all copies.
 -
 - THE SOFTWARE IS PROVIDED "AS IS" AND ISC DISCLAIMS ALL WARRANTIES WITH
 - REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY
 - AND FITNESS.  IN NO EVENT SHALL ISC BE LIABLE FOR ANY SPECIAL, DIRECT,
 - INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM
 - LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE
 - OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
 - PERFORMANCE OF THIS SOFTWARE.
-->
# BIND 9

### Contents

1. [Introduction](#intro)
1. [Reporting bugs and getting help](#help)
1. [Contributing to BIND](#contrib)
1. [BIND 9.10 features](#features)
1. [Building BIND](#build)
1. [macOS](#macos)
1. [Compile-time options](#opts)
1. [Automated testing](#testing)
1. [Documentation](#doc)
1. [Change log](#changes)
1. [Acknowledgments](#ack)

### <a name="intro"/> Introduction

BIND (Berkeley Internet Name Domain) is a complete, highly portable
implementation of the DNS (Domain Name System) protocol.

The BIND name server, `named`, is able to serve as an authoritative name
server, recursive resolver, DNS forwarder, or all three simultaneously.  It
implements views for split-horizon DNS, automatic DNSSEC zone signing and
key management, catalog zones to facilitate provisioning of zone data
throughout a name server constellation, response policy zones (RPZ) to
protect clients from malicious data, response rate limiting (RRL) and
recursive query limits to reduce distributed denial of service attacks,
and many other advanced DNS features.  BIND also includes a suite of
administrative tools, including the `dig` and `delv` DNS lookup tools,
`nsupdate` for dynamic DNS zone updates, `rndc` for remote name server
administration, and more.

BIND 9 is a complete re-write of the BIND architecture that was used in
versions 4 and 8.  Internet Systems Consortium
([https://www.isc.org](https://www.isc.org)), a 501(c)(3) public benefit
corporation dedicated to providing software and services in support of the
Internet infrastructure, developed BIND 9 and is responsible for its
ongoing maintenance and improvement.  BIND is open source software
licenced under the terms of the ISC License for all versions up to and
including BIND 9.10, and the Mozilla Public License version 2.0 for all
subsequent verisons.

For a summary of features introduced in past major releases of BIND,
see the file [HISTORY](HISTORY.md).

For a detailed list of changes made throughout the history of BIND 9, see
the file [CHANGES](CHANGES). See [below](#changes) for details on the
CHANGES file format.

For up-to-date release notes and errata, see
[http://www.isc.org/software/bind9/releasenotes](http://www.isc.org/software/bind9/releasenotes)

### <a name="help"/> Reporting bugs and getting help

To report non-security-sensitive bugs or request new features, you may
open an Issue in the BIND 9 project on the
[ISC GitLab server](https://gitlab.isc.org) at
[https://gitlab.isc.org/isc-projects/bind9](https://gitlab.isc.org/isc-projects/bind9).

Please note that, unless you explicitly mark the newly created Issue as
"confidential", it will be publicly readable.  Please do not include any
information in bug reports that you consider to be confidential unless
the issue has been marked as such.  In particular, if submitting the
contents of your configuration file in a non-confidential Issue, it is
advisable to obscure key secrets: this can be done automatically by
using `named-checkconf -px`.

If the bug you are reporting is a potential security issue, such as an
assertion failure or other crash in `named`, please do *NOT* use GitLab to
report it. Instead, please send mail to
[security-officer@isc.org](mailto:security-officer@isc.org).

Professional support and training for BIND are available from
ISC at [https://www.isc.org/support](https://www.isc.org/support).

To join the __BIND Users__ mailing list, or view the archives, visit
[https://lists.isc.org/mailman/listinfo/bind-users](https://lists.isc.org/mailman/listinfo/bind-users).

If you're planning on making changes to the BIND 9 source code, you
may also want to join the __BIND Workers__ mailing list, at
[https://lists.isc.org/mailman/listinfo/bind-workers](https://lists.isc.org/mailman/listinfo/bind-workers).

### <a name="contrib"/> Contributing to BIND

ISC maintains a public git repository for BIND; details can be found
at [http://www.isc.org/git/](http://www.isc.org/git/), and also on Github
at [https://github.com/isc-projects](https://github.com/isc-projects).

Information for BIND contributors can be found in the following files:
- General information: [doc/dev/contrib.md](doc/dev/contrib.md)
- BIND 9 code style: [doc/dev/style.md](doc/dev/style.md)
- BIND architecture and developer guide: [doc/dev/dev.md](doc/dev/dev.md)

Patches for BIND may be submitted as
[Merge Requests](https://gitlab.isc.org/isc-projects/bind9/merge_requests)
in the [ISC GitLab server](https://gitlab.isc.org) at
at [https://gitlab.isc.org/isc-projects/bind9/merge_requests](https://gitlab.isc.org/isc-projects/bind9/merge_requests).

By default, external contributors don't have ability to fork BIND in the
GitLab server, but if you wish to contribute code to BIND, you may request
permission to do so. Thereafter, you can create git branches and directly
submit requests that they be reviewed and merged.

If you prefer, you may also submit code by opening a
[GitLab Issue](https://gitlab.isc.org/isc-projects/bind9/issues) and
including your patch as an attachment, preferably generated by
`git format-patch`.

### <a name="features"/> BIND 9.10 features

BIND 9.10.0 includes a number of changes from BIND 9.9 and earlier
releases.  New features include:

 * DNS Response-rate limiting (DNS RRL), which blunts the impact of
   reflection and amplification attacks, is always compiled in and no
   longer requires a compile-time option to enable it.
 * An experimental "Source Identity Token" (SIT) EDNS option is now
   available.  Similar to DNS Cookies as invented by Donald Eastlake 3rd,
   these are designed to enable clients to detect off-path spoofed
   responses, and to enable servers to detect spoofed-source queries.
   Servers can be configured to send smaller responses to clients that have
   not identified themselves using a SIT option, reducing the effectiveness
   of amplification attacks.  RRL processing has also been updated; clients
   proven to be legitimate via SIT are not subject to rate limiting.  Use
   `configure --enable-sit` to enable this feature in BIND.
 * A new zone file format, `map`, stores zone data in a format that can be
   mapped directly into memory, allowing significantly faster zone loading.
 * `delv` (domain entity lookup and validation) is a new tool with dig-like
   semantics for looking up DNS data and performing internal DNSSEC
   validation.  This allows easy validation in environments where the
   resolver may not be trustworthy, and assists with troubleshooting of
   DNSSEC problems. (NOTE: In previous development releases of BIND 9.10,
   this utility was called `delve`. The spelling has been changed to avoid
   confusion with the `delve` utility included with the Xapian search
   engine.)
 * Improved EDNS(0) processing for better resolver performance and
   reliability over slow or lossy connections.
 * A new `configure --with-tuning=large` option tunes certain compiled-in
   constants and default settings to values better suited to large servers
   with abundant memory.  This can improve performance on such servers, but
   will consume more memory and may degrade performance on smaller systems.
 * Substantial improvement in response-policy zone (RPZ) performance.  Up
   to 32 response-policy zones can be configured with minimal performance
   loss.
 * To improve recursive resolver performance, cache records which are still
   being requested by clients can now be automatically refreshed from the
   authoritative server before they expire, reducing or eliminating the
   time window in which no answer is available in the cache.
 * New `rpz-client-ip` triggers and drop policies allowing response
   policies based on the IP address of the client.
 * ACLs can now be specified based on geographic location using the MaxMind
   GeoIP databases.  Use `configure --with-geoip` to enable.
 * Zone data can now be shared between views, allowing multiple views to
   serve the same zones authoritatively without storing multiple copies in
   memory.
 * New XML schema (version 3) for the statistics channel includes many new
   statistics and uses a flattened XML tree for faster parsing. The older
   schema is now deprecated.
 * A new stylesheet, based on the Google Charts API, displays XML
   statistics in charts and graphs on javascript-enabled browsers.
 * The statistics channel can now provide data in JSON format as well as
   XML.
 * New stats counters track TCP and UDP queries received per zone, and EDNS
   options received in total.
 * The internal and export versions of the BIND libraries (libisc, libdns,
   etc) have been unified so that external library clients can use the same
   libraries as BIND itself.
 * A new compile-time option, `configure --enable-native-pkcs11`, allows
   BIND 9 cryptography functions to use the PKCS#11 API natively, so that
   BIND can drive a cryptographic hardware service module (HSM) directly
   instead of using a modified OpenSSL as an intermediary. (Note: This
   feature requires an HSM to have a full implementation of the PKCS#11
   API; many current HSMs only have partial implementations. The new
   `pkcs11-tokens` command can be used to check API completeness.  Native
   PKCS#11 is known to work with the Thales nShield HSM and with SoftHSM
   version 2 from the Open DNSSEC project.)
 * The new `max-zone-ttl` option enforces maximum TTLs for zones. This can
   simplify the process of rolling DNSSEC keys by guaranteeing that cached
   signatures will have expired within the specified amount of time.
 * `dig +subnet` sends an EDNS CLIENT-SUBNET option when querying.
 * `dig +expire` sends an EDNS EXPIRE option when querying.  When this
   option is sent with an SOA query to a server that supports it, it will
   report the expiry time of a slave zone.
 * New `dnssec-coverage` tool to check DNSSEC key coverage for a zone and
   report if a lapse in signing coverage has been inadvertently scheduled.
 * Signing algorithm flexibility and other improvements for the `rndc`
   control channel.
 * `named-checkzone` and `named-compilezone` can now read journal files,
   allowing them to process dynamic zones.
 * Multiple DLZ databases can now be configured.  Individual zones can be
   configured to be served from a specific DLZ database.  DLZ databases now
   serve zones of type `master` and `redirect`.
 * `rndc zonestatus` reports information about a specified zone.
 * `named` now listens on IPv6 as well as IPv4 interfaces by default.
 * `named` now preserves the capitalization of names when responding to
   queries: for instance, a query for "example.com" may be answered with
   "example.COM" if the name was configured that way in the zone file.
   Some clients have a bug causing them to depend on the older behavior, in
   which the case of the answer always matched the case of the query,
   rather than the case of the name configured in the DNS.  Such clients
   can now be specified in the new `no-case-compress` ACL; this will
   restore the older behavior of `named` for those clients only.
 * new `dnssec-importkey` command allows the use of offline DNSSEC keys
   with automatic DNSKEY management.
 * New `named-rrchecker` tool to verify the syntactic correctness of
   individual resource records.
 * When re-signing a zone, the new `dnssec-signzone -Q` option drops
   signatures from keys that are still published but are no longer active.
 * `named-checkconf -px` will print the contents of configuration files
   with the shared secrets obscured, making it easier to share
   configuration (e.g. when submitting a bug report) without revealing
   private information.
 * `rndc scan` causes named to re-scan network interfaces for changes in
   local addresses.
 * On operating systems with support for routing sockets, network
   interfaces are re-scanned automatically whenever they change.
 * `tsig-keygen` is now available as an alternate command name to use for
   `ddns-confgen`.

#### BIND 9.10.1

BIND 9.10.1 is a maintenance release, and addresses the security flaws
described in CVE-2014-3214 and CVE-2014-3859.

#### BIND 9.10.2

BIND 9.10.2 is a maintenance release, and addresses the security flaws
described in CVE-2014-8500, CVE-2014-8680 and CVE-2015-1349.

#### BIND 9.10.3

BIND 9.10.3 is a maintenance release, and addresses the security flaws
described in CVE-2015-4620, CVE-2015-5477, CVE-2015-5722, and
CVE-2015-5986.

It also makes the following new features available:

* New "fetchlimit" quotas are now available for the use of
  recursive resolvers that are are under high query load for
  domains whose authoritative servers are nonresponsive or are
  experiencing a denial of service attack.

    * `fetches-per-server` limits the number of simultaneous queries that
      can be sent to any single authoritative server.  The configured value
      is a starting point; it is automatically adjusted downward if the
      server is partially or completely non-responsive. The algorithm used
      to adjust the quota can be configured via the `fetch-quota-params`
      option.
    * `fetches-per-zone` limits the number of simultaneous queries that can
      be sent for names within a single domain.  (Note: Unlike
      `fetches-per-server`, this value is not self-tuning.)
    * New stats counters have been added to count queries spilled due to
      these quotas.

  NOTE: These features are NOT built in by default; use
  `configure --enable-fetchlimit` to enable them.

* `dig` now supports sending of arbitrary EDNS options by specifying
  them on the command line.

#### BIND 9.10.4

BIND 9.10.4 is a maintenance release, and addresses the security flaws
described in CVE-2015-8000, CVE-2015-8461, CVE-2015-8704, CVE-2015-8705,
CVE-2016-1285, CVE-2016-1286, CVE-2016-2088, CVE-2016-2775 and
CVE-2016-2776.

#### BIND 9.10.5
	
BIND 9.10.5 is a maintenance release, and addresses the security flaws
disclosed in CVE-2016-2775, CVE-2016-2776, CVE-2016-6170, CVE-2016-8864,
CVE-2016-9131, CVE-2016-9147, CVE-2016-9444, CVE-2017-3135, CVE-2017-3136,
CVE-2017-3137, and CVE-2017-3138.

#### BIND 9.10.6

BIND 9.10.6 is a maintenance release, and addresses the security
flaws disclosed in CVE-2017-3140 and CVE-2017-3141, CVE-2017-3142
and CVE-2017-3143.

#### BIND 9.10.7

BIND 9.10.7 is a maintenance release, and addresses the security
flaw disclosed in CVE-2017-3145.

### <a name="build"/> Building BIND

BIND requires a UNIX or Linux system with an ANSI C compiler, basic POSIX
support, and a 64-bit integer type. Successful builds have been observed on
many versions of Linux and UNIX, including RedHat, Fedora, Debian, Ubuntu,
SuSE, Slackware, FreeBSD, NetBSD, OpenBSD, Mac OS X, Solaris, HP-UX, AIX,
SCO OpenServer, and OpenWRT. 

BIND is also available for Windows XP, 2003, 2008, and higher.  See
`win32utils/readme1st.txt` for details on building for Windows systems.

To build on a UNIX or Linux system, use:

		$ ./configure
		$ make

If you're planning on making changes to the BIND 9 source, you should run
`make depend`.  If you're using Emacs, you might find `make tags` helpful.

Several environment variables that can be set before running `configure` will
affect compilation:

|Variable|Description |
|--------------------|-----------------------------------------------|
|`CC`|The C compiler to use.  `configure` tries to figure out the right one for supported systems.|
|`CFLAGS`|C compiler flags.  Defaults to include -g and/or -O2 as supported by the compiler.  Please include '-g' if you need to set `CFLAGS`. |
|`STD_CINCLUDES`|System header file directories.  Can be used to specify where add-on thread or IPv6 support is, for example.  Defaults to empty string.|
|`STD_CDEFINES`|Any additional preprocessor symbols you want defined.  Defaults to empty string. For a list of possible settings, see the file [OPTIONS](OPTIONS.md).|
|`LDFLAGS`|Linker flags. Defaults to empty string.|
|`BUILD_CC`|Needed when cross-compiling: the native C compiler to use when building for the target system.|
|`BUILD_CFLAGS`|Optional, used for cross-compiling|
|`BUILD_CPPFLAGS`||
|`BUILD_LDFLAGS`||
|`BUILD_LIBS`||

#### <a name="macos"> macOS

Building on macOS assumes that the "Command Tools for Xcode" is installed.
This can be downloaded from https://developer.apple.com/download/more/
or if you have Xcode already installed you can run "xcode-select --install".
This will add /usr/include to the system and install the compiler and other
tools so that they can be easily found.


#### <a name="opts"/> Compile-time options

To see a full list of configuration options, run `configure --help`.

On most platforms, BIND 9 is built with multithreading support, allowing it
to take advantage of multiple CPUs.  You can configure this by specifying
`--enable-threads` or `--disable-threads` on the `configure` command line.
The default is to enable threads, except on some older operating systems on
which threads are known to have had problems in the past.  (Note: Prior to
BIND 9.10, the default was to disable threads on Linux systems; this has
now been reversed.  On Linux systems, the threaded build is known to change
BIND's behavior with respect to file permissions; it may be necessary to
specify a user with the -u option when running `named`.)

To build shared libraries, specify `--with-libtool` on the `configure`
command line.

Certain compiled-in constants and default settings can be increased to
values better suited to large servers with abundant memory resources (e.g,
64-bit servers with 12G or more of memory) by specifying
`--with-tuning=large` on the `configure` command line. This can improve
performance on big servers, but will consume more memory and may degrade
performance on smaller systems.

For the server to support DNSSEC, you need to build it with crypto support.
To use OpenSSL, you should have OpenSSL 1.0.2e or newer installed.  If the
OpenSSL library is installed in a nonstandard location, specify the prefix
using "--with-openssl=&lt;PREFIX&gt;" on the configure command line. To use a
PKCS#11 hardware service module for cryptographic operations, specify the
path to the PKCS#11 provider library using "--with-pkcs11=&lt;PREFIX&gt;", and
configure BIND with "--enable-native-pkcs11".

To support the HTTP statistics channel, the server must be linked with at
least one of the following: libxml2
[http://xmlsoft.org](http://xmlsoft.org) or json-c
[https://github.com/json-c](https://github.com/json-c).  If these are
installed at a nonstandard location, specify the prefix using
`--with-libxml2=/prefix` or `--with-libjson=/prefix`.

To support GeoIP location-based ACLs, the server must be linked with
libGeoIP. This is not turned on by default; BIND must be configured with
"--with-geoip". If the library is installed in a nonstandard location, use
specify the prefix using "--with-geoip=/prefix".

Portions of BIND that are written in Python, including
`dnssec-coverage`, `dnssec-checkds`, and some of the
system tests, require the 'argparse' module to be available.
'argparse' is a standard module as of Python 2.7 and Python 3.2.

On some platforms it is necessary to explicitly request large file support
to handle files bigger than 2GB.  This can be done by using
`--enable-largefile` on the `configure` command line.

Support for the "fixed" rrset-order option can be enabled or disabled by
specifying `--enable-fixed-rrset` or `--disable-fixed-rrset` on the
configure command line.  By default, fixed rrset-order is disabled to
reduce memory footprint.

If your operating system has integrated support for IPv6, it will be used
automatically.  If you have installed KAME IPv6 separately, use
`--with-kame[=PATH]` to specify its location.

`make install` will install `named` and the various BIND 9 libraries.  By
default, installation is into /usr/local, but this can be changed with the
`--prefix` option when running `configure`.

You may specify the option `--sysconfdir` to set the directory where
configuration files like `named.conf` go by default, and `--localstatedir`
to set the default parent directory of `run/named.pid`.   For backwards
compatibility with BIND 8, `--sysconfdir` defaults to `/etc` and
`--localstatedir` defaults to `/var` if no `--prefix` option is given.  If
there is a `--prefix` option, sysconfdir defaults to `$prefix/etc` and
localstatedir defaults to `$prefix/var`.

### <a name="testing"/> Automated testing

A system test suite can be run with `make test`.  The system tests require
you to configure a set of virtual IP addresses on your system (this allows
multiple servers to run locally and communicate with one another).  These
IP addresses can be configured by running the command
`bin/tests/system/ifconfig.sh up` as root.

Some tests require Perl and the Net::DNS and/or IO::Socket::INET6 modules,
and will be skipped if these are not available. Some tests require Python
and the 'dnspython' module and will be skipped if these are not available.
See bin/tests/system/README for further details.

Unit tests are implemented using Automated Testing Framework (ATF).
To run them, use `configure --with-atf`, then run `make test` or
`make unit`.

### <a name="doc"/> Documentation

The *BIND 9 Administrator Reference Manual* is included with the source
distribution, in DocBook XML, HTML and PDF format, in the `doc/arm`
directory.

Some of the programs in the BIND 9 distribution have man pages in their
directories.  In particular, the command line options of `named` are
documented in `bin/named/named.8`.

Frequently (and not-so-frequently) asked questions and their answers
can be found in the ISC Knowledge Base at
[https://kb.isc.org](https://kb.isc.org).

Additional information on various subjects can be found in other
`README` files throughout the source tree.

### <a name="changes"/> Change log

A detailed list of all changes that have been made throughout the
development BIND 9 is included in the file CHANGES, with the most recent
changes listed first.  Change notes include tags indicating the category of
the change that was made; these categories are:

|Category	|Description	        			|
|--------------	|-----------------------------------------------|
| [func] | New feature |
| [bug] | General bug fix |
| [security] | Fix for a significant security flaw |
| [experimental] | Used for new features when the syntax or other aspects of the design are still in flux and may change |
| [port] | Portability enhancement |
| [maint] | Updates to built-in data such as root server addresses and keys |
| [tuning] | Changes to built-in configuration defaults and constants to improve performance |
| [performance] | Other changes to improve server performance |
| [protocol] | Updates to the DNS protocol such as new RR types |
| [test] | Changes to the automatic tests, not affecting server functionality |
| [cleanup] | Minor corrections and refactoring |
| [doc] | Documentation |
| [contrib] | Changes to the contributed tools and libraries in the 'contrib' subdirectory |
| [placeholder] | Used in the master development branch to reserve change numbers for use in other branches, e.g. when fixing a bug that only exists in older releases |

In general, [func] and [experimental] tags will only appear in new-feature
releases (i.e., those with version numbers ending in zero).  Some new
functionality may be backported to older releases on a case-by-case basis.
All other change types may be applied to all currently-supported releases.

### <a name="ack"/> Acknowledgments

* The original development of BIND 9 was underwritten by the
  following organizations:

		Sun Microsystems, Inc.
		Hewlett Packard
		Compaq Computer Corporation
		IBM
		Process Software Corporation
		Silicon Graphics, Inc.
		Network Associates, Inc.
		U.S. Defense Information Systems Agency
		USENIX Association
		Stichting NLnet - NLnet Foundation
		Nominum, Inc.

* This product includes software developed by the OpenSSL Project for use
  in the OpenSSL Toolkit.
  [http://www.OpenSSL.org/](http://www.OpenSSL.org/)
* This product includes cryptographic software written by Eric Young
  (eay@cryptsoft.com)
* This product includes software written by Tim Hudson (tjh@cryptsoft.com)
