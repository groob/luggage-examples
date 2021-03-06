#
#   Copyright 2009 Joe Block <jpb@ApesSeekingKnowledge.net>
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
#
#  Example:  A dynamic installer for Marionette-Collective
#
#  Author:  Gary Larizza
#  Created: 12/17/2010
#  Last Modified: 8/1/2011
#
#  Description:  This Makefile will download the version of MCollective
#    specified in the PACKAGE_VERSION variable from the Puppet Labs website,
#    untar it, and then install the source files into their Mac-specific
#    locations. The MAJOR and MINOR versions must be specified for the
#    Info.plist file that Packagemaker requires, but I use awk on the
#    PACKAGE_VERSION to get these.  See inline comments.
#
USE_PKGBUILD=1
include /usr/local/share/luggage/luggage.make


# Luggage Variables:
#    If the TYPE variable isn't specified via the CLI, we will install
#    everything into the resultant package
TITLE=MCollective_Installer_Full
REVERSE_DOMAIN=com.puppetlabs
PAYLOAD=\
  	unpack-mc-${MCFILE} \
		pack-mc-libexec \
		pack-mc-binaries \
		pack-mc-lib \
		pack-mc-config \
		pack-mc-config-server \
		pack-mc-config-client \
		pack-mc-launchd \
		pack-mc-mcollectived \
		pack-mc-preflight-all \
		# pack-script-postinstall

# Variable Declarations:  
#    Any variable can be set from the command line by doing this:
#    "make pkg PACKAGE_VERSION=1.2.1"
PACKAGE_VERSION=2.3.2
PACKAGE_MAJOR_VERSION=`echo ${PACKAGE_VERSION} | awk -F '.' '{print $$1}'`
PACKAGE_MINOR_VERSION=`echo ${PACKAGE_VERSION} | awk -F '.' '{print $$2$$3}'`
PACKAGE_VERSION_CHECK=`echo ${PACKAGE_VERSION} | awk -F '.' '{print $$3}'`
PACKAGE_VERSION_FULL=`echo ${PACKAGE_VERSION} | awk -F '.' '{print $$1$$2$$3}'`
MCFILE=mcollective-${PACKAGE_VERSION}
MCURL=http://downloads.puppetlabs.com/mcollective/${MCFILE}.tgz

# Package Creation Limiters:
#    These if-statements will check for one of three values for the TYPE 
#    "variable: COMMON, CLIENT, or BASE"  If either of these values are present
#     (CASE SENSITIVE) the PAYLOAD variable will be changed to limit what is 
#     installed into the package.

# COMMON Package:
#    This package includes the Ruby libraries and MCollective plugins with 
#    nothing else.

ifeq (${TYPE},COMMON)
	PAYLOAD=\
			unpack-mc-${MCFILE} \
			pack-mc-libexec \
			pack-mc-lib \
			pack-mc-preflight-common
	TITLE=MCollective_Installer_Common
endif

# CLIENT Package:
#    This package includes the MCollective Binaries and the configuration file 
#    for MCollective's client binaries.
ifeq (${TYPE},CLIENT)
	PAYLOAD=\
			unpack-mc-${MCFILE} \
			pack-mc-config \
			pack-mc-binaries \
			pack-mc-config-client \
			pack-mc-preflight-client
	TITLE=MCollective_Installer_Client
endif

# BASE Package:
#    This package includes the mcollectived daemon, Ruby Libraries, a launchd 
#    plist to call mcollectived, and the configuration files for the MCollective
#    server.

ifeq (${TYPE},BASE)
	PAYLOAD=\
			unpack-mc-${MCFILE} \
			pack-mc-config \
			pack-mc-config-server \
			pack-mc-launchd \
			pack-mc-mcollectived \
			pack-mc-lib \
			pack-mc-preflight-base \
			# pack-script-postinstall
	TITLE=MCollective_Installer_Base
endif

# This rule will curl the selected version of MCollective and untar it into the 
#    directory in which the Makefile resides.

l_etc_mcollective: l_private_etc
	@sudo mkdir -p ${WORK_D}/private/etc/mcollective
	@sudo chown -R root:wheel ${WORK_D}/private/etc/mcollective
	@sudo chmod -R 755 ${WORK_D}/private/etc/mcollective

l_usr_libexec: l_usr
	@sudo mkdir -p ${WORK_D}/usr/libexec
	@sudo chown -R root:wheel ${WORK_D}/usr/libexec
	@sudo chmod -R 755 ${WORK_D}/usr/libexec

l_usr_libexec_mcollective: l_usr
	@sudo mkdir -p ${WORK_D}/usr/libexec/mcollective
	@sudo chown -R root:wheel ${WORK_D}/usr/libexec/mcollective
	@sudo chmod -R 755 ${WORK_D}/usr/libexec/mcollective

# l_usr_lib_ruby_site_ruby_1_8: l_usr
# 	@sudo mkdir -p ${WORK_D}/usr/lib/ruby/site_ruby/1.8
# 	@sudo chown -R root:wheel ${WORK_D}/usr/lib/ruby/site_ruby/1.8
# 	@sudo chmod -R 755 ${WORK_D}/usr/lib/ruby/site_ruby/1.8
l_usr_lib_ruby_site_ruby_2_0_0: l_usr
	@sudo mkdir -p ${WORK_D}/usr/lib/ruby/site_ruby/2.0.0
	@sudo chown -R root:wheel ${WORK_D}/usr/lib/ruby/site_ruby/2.0.0
	@sudo chmod -R 755 ${WORK_D}/usr/lib/ruby/site_ruby/2.0.0

unpack-mc-${MCFILE}:
	curl ${MCURL} -o ${MCFILE}.tgz
	@sudo ${TAR} xvzf ${MCFILE}.tgz

# This rule will install MCollective's plugin files to /usr/libexec/mcollective
pack-mc-libexec: l_usr_libexec_mcollective
	@sudo ${CP} -R ./${MCFILE}/plugins/* ${WORK_D}/usr/libexec/mcollective/
	@sudo chmod -R 755 ${WORK_D}/usr/libexec/mcollective

# This rule installs the MCollective client binaries to /usr/sbin
pack-mc-binaries: l_usr_sbin
	@sudo ${INSTALL} -m 755 ./${MCFILE}/bin/mc-* ${WORK_D}/usr/sbin
	@sudo ${INSTALL} -m 755 ./${MCFILE}/bin/mco ${WORK_D}/usr/sbin
	# if [ ${PACKAGE_VERSION_FULL} -gt 113 ]; then /usr/bin/sudo ${INSTALL} \
  -m 755 ./${MCFILE}/bin/mco ${WORK_D}/usr/sbin; fi

# This rule will install the mcollectived daemon file to /usr/sbin
pack-mc-mcollectived: l_usr_sbin
	@sudo ${INSTALL} -m 755 ./${MCFILE}/bin/mcollectived ${WORK_D}/usr/sbin/mcollectived

# This rule installs the MCollective Ruby Libraries to 
# /usr/lib/ruby/site_ruby/1.8
# pack-mc-lib: l_usr_lib_ruby_site_ruby_1_8
# 	@sudo ${CP} -R ./${MCFILE}/lib/* ${WORK_D}/usr/lib/ruby/site_ruby/1.8
# 	@sudo chmod -R 755 ${WORK_D}/usr/lib/ruby/site_ruby/1.8
pack-mc-lib: l_usr_lib_ruby_site_ruby_2_0_0
	@sudo ${CP} -R ./${MCFILE}/lib/* ${WORK_D}/usr/lib/ruby/site_ruby/2.0.0
	@sudo chmod -R 755 ${WORK_D}/usr/lib/ruby/site_ruby/2.0.0

# This rule will install base configuration files to /etc/mcollective
pack-mc-config: l_etc_mcollective
	@sudo ${INSTALL} -m 755 ./${MCFILE}/etc/facts.yaml.dist \
		${WORK_D}/private/etc/mcollective/facts.yaml
	@sudo ${INSTALL} -m 755 ./${MCFILE}/etc/rpc-help.erb \
		${WORK_D}/private/etc/mcollective/rpc-help.erb
	@sudo ${CP} -R ./${MCFILE}/etc/ssl ${WORK_D}/private/etc/mcollective/
	@sudo chmod -R 755 ${WORK_D}/private/etc/mcollective

# This rule installs the MCollective server configuration file to /etc/mcollective
#    It also adjusts the daemonize setting for OS X
pack-mc-config-server: l_etc_mcollective
	@sudo ${INSTALL} -m 600 ./${MCFILE}/etc/server.cfg.dist \
		${WORK_D}/private/etc/mcollective/server.cfg
	sed -i '' "s#daemonize = 1#daemonize = 0#g" "${WORK_D}/private/etc/mcollective/server.cfg"

# This rule installs the MCollective server configuration file to 
#    /private/etc/mcollective  It also adjusts the libdir setting to reflect the
#    directory we've used above.

pack-mc-config-client: l_etc_mcollective
	@sudo ${INSTALL} -m 600 ./${MCFILE}/etc/client.cfg.dist \
		${WORK_D}/private/etc/mcollective/client.cfg

# This rule installes a launchd plist to call mcollectived into 
#   /Library/LaunchDaemons 
pack-mc-launchd: l_Library_LaunchDaemons
	@sudo ${INSTALL} -m 644 ./com.puppetlabs.mcollective.plist \
		${WORK_D}/Library/LaunchDaemons

# Preflight Settings:
#    The MCollective preflight scripts will uninstall current versions of 
#    files that the package intends to install. This allows us to upgrade or 
#    roll-back versions of MCollective files. Since we install different files 
#    depending on the TYPE variable, we need different versions of the 
#    preflight script.

pack-mc-preflight-all:
	@sudo ${INSTALL} -m 755 ./preflight ${SCRIPT_D}

pack-mc-preflight-client:
	@sudo ${INSTALL} -m 755 ./preflight_client ${SCRIPT_D}/preflight

pack-mc-preflight-common:
	@sudo ${INSTALL} -m 755 ./preflight_common ${SCRIPT_D}/preflight

pack-mc-preflight-base:
	@sudo ${INSTALL} -m 755 ./preflight_base ${SCRIPT_D}/preflight

