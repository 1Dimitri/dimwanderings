---
id: 623
title: 'Domain Join Error 0x6ba'
date: '2017-03-14T15:39:50+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=623'
permalink: /2017/03/14/domain-join-error-0x6ba/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"625";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/03/domain-join-error.jpg
categories:
    - 'Active Directory'
    - troubleshooting
tags:
    - 'Active Directory'
    - 'domain join'
    - error
    - netsetup
    - troubleshooting
---

While joining machine to a domain, multiple errors can occur. In particular Domain Join Error 0x6ba is often present but with [different causes](https://social.technet.microsoft.com/Forums/windowsserver/en-US/919f0a99-25ff-448c-99a4-a29ba6ae16e9/while-joining-domain-pc-gives-error-rpc-server-unavilable?forum=winservergen).

Let’s recap the needed things to properly join the domain:

1. The machine must be able to solve the domain name
2. The machine must be able to retrieve a domain controller name from that domain
3. The machine must be able to contact that Domain Controller
4. The user who is performing the action must have enough rights to create or update the computer account in that domain
5. but the domain controller must also be able to contact that machine

Sometimes this cheat-sheet is not enough and you’re wondering why the join operation fails, as most of the times the error is very generic.

Please be reminded that in C:\\Windows\\Debug there’s a NetSetup.log which traces back every attempt to join or unjoin a domain.

Let’s see a somewhat advanced example where the machine starts correctly but when it reboots the domain administrators cannot log in

```
03/01/2017 10:22:34:797 -----------------------------------------------------------------
03/01/2017 10:22:34:797 NetpDoDomainJoin
03/01/2017 10:22:34:797 NetpDoDomainJoin: using current computer names
03/01/2017 10:22:34:797 NetpDoDomainJoin: NetpGetComputerNameEx(NetBios) returned 0x0
03/01/2017 10:22:34:797 NetpDoDomainJoin: NetpGetComputerNameEx(DnsHostName) returned 0x0
03/01/2017 10:22:34:797 NetpMachineValidToJoin: 'myserver'
03/01/2017 10:22:34:797 	OS Version: 6.3
03/01/2017 10:22:34:797 	Build number: 9600 (9600.winblue_rtm.130821-1623)
03/01/2017 10:22:34:797 	SKU: Windows Server 2012 R2 Standard
03/01/2017 10:22:34:797 	Architecture: 64-bit (AMD64)
03/01/2017 10:22:34:797 NetpDomainJoinLicensingCheck: ulLicenseValue=1, Status: 0x0
03/01/2017 10:22:34:797 NetpGetLsaPrimaryDomain: status: 0x0
03/01/2017 10:22:34:797 NetpMachineValidToJoin: status: 0x0
03/01/2017 10:22:34:797 NetpJoinDomain
03/01/2017 10:22:34:797 	HostName: myserver
03/01/2017 10:22:34:797 	NetbiosName: myserver
03/01/2017 10:22:34:797 	Domain: mydomain.local
03/01/2017 10:22:34:797 	MachineAccountOU: (NULL)
03/01/2017 10:22:34:797 	Account: mydomain.local\AdminUser
03/01/2017 10:22:34:797 	Options: 0x27
03/01/2017 10:22:34:797 NetpLoadParameters: loading registry parameters...
03/01/2017 10:22:34:797 NetpLoadParameters: DNSNameResolutionRequired not found, defaulting to '1' 0x2
03/01/2017 10:22:34:797 NetpLoadParameters: DomainCompatibilityMode not found, defaulting to '0' 0x2
03/01/2017 10:22:34:797 NetpLoadParameters: status: 0x2
03/01/2017 10:22:34:797 NetpValidateName: checking to see if 'mydomain.local' is valid as type 3 name
03/01/2017 10:22:34:875 NetpCheckDomainNameIsValid [ Exists ] for 'mydomain.local' returned 0x0
03/01/2017 10:22:34:875 NetpValidateName: name 'mydomain.local' is valid for type 3
03/01/2017 10:22:34:875 NetpDsGetDcName: trying to find DC in domain 'mydomain.local', flags: 0x40001010
03/01/2017 10:22:35:750 NetpDsGetDcName: failed to find a DC having account 'myserver$': 0x525, last error is 0x0
03/01/2017 10:22:35:812 NetpLoadParameters: loading registry parameters...
03/01/2017 10:22:35:812 NetpLoadParameters: DNSNameResolutionRequired not found, defaulting to '1' 0x2
03/01/2017 10:22:35:812 NetpLoadParameters: DomainCompatibilityMode not found, defaulting to '0' 0x2
03/01/2017 10:22:35:812 NetpLoadParameters: status: 0x2
03/01/2017 10:22:35:812 NetpDsGetDcName: status of verifying DNS A record name resolution for 'localdc02.mydomain.local': 0x0
03/01/2017 10:22:35:812 NetpDsGetDcName: found DC '\\localdc02.mydomain.local' in the specified domain
03/01/2017 10:22:35:812 NetpJoinDomainOnDs: NetpDsGetDcName returned: 0x0
03/01/2017 10:22:35:812 NetpDisableIDNEncoding: using FQDN mydomain.local from dcinfo
03/01/2017 10:22:35:812 NetpDisableIDNEncoding: DnsDisableIdnEncoding(UNTILREBOOT) on 'mydomain.local' succeeded
03/01/2017 10:22:35:812 NetpJoinDomainOnDs: NetpDisableIDNEncoding returned: 0x0
03/01/2017 10:22:35:812 NetpJoinDomainOnDs: status of connecting to dc '\\localdc02.mydomain.local': 0x0
03/01/2017 10:22:35:812 NetpGetDnsHostName: PrimaryDnsSuffix defaulted to DNS domain name: mydomain.local
03/01/2017 10:22:35:812 NetpProvisionComputerAccount:
03/01/2017 10:22:35:812 	lpDomain: mydomain.local
03/01/2017 10:22:35:812 	lpHostName: myserver
03/01/2017 10:22:35:812 	lpMachineAccountOU: (NULL)
03/01/2017 10:22:35:812 	lpDcName: localdc02.mydomain.local
03/01/2017 10:22:35:812 	lpMachinePassword: (null)
03/01/2017 10:22:35:812 	lpAccount: mydomain.local\AdminUser
03/01/2017 10:22:35:812 	lpPassword: (non-null)
03/01/2017 10:22:35:812 	dwJoinOptions: 0x27
03/01/2017 10:22:35:812 	dwOptions: 0x40000003
03/01/2017 10:22:35:828 NetpLdapBind: Verified minimum encryption strength on localdc02.mydomain.local: 0x0
03/01/2017 10:22:35:828 NetpLdapGetLsaPrimaryDomain: reading domain data
03/01/2017 10:22:35:828 NetpGetNCData: Reading NC data
03/01/2017 10:22:35:828 NetpGetDomainData: Lookup domain data for: DC=mydomain,DC=local
03/01/2017 10:22:35:844 NetpGetDomainData: Lookup crossref data for: CN=Partitions,CN=Configuration,DC=mydomain,DC=local
03/01/2017 10:22:35:844 NetpLdapGetLsaPrimaryDomain: result of retrieving domain data: 0x0
03/01/2017 10:22:35:844 NetpGetLocalDACDisabled: returning 0x0, *pfDACDisabled=TRUE
03/01/2017 10:22:35:844 NetpCheckForDomainSIDCollision: returning 0x0(0).
03/01/2017 10:22:40:281 [00000470] NetpGetLsaPrimaryDomain: status: 0x0
03/01/2017 10:22:40:281 [000004e4] NetpGetLsaPrimaryDomain: status: 0x0
03/01/2017 10:22:56:859 NetpGetComputerObjectDn: Unable to bind to DS on '\\localdc02.mydomain.local': 0x6ba
03/01/2017 10:22:56:859 NetpCreateComputerObjectInDs: NetpGetComputerObjectDn failed: 0x6ba
03/01/2017 10:22:56:859 NetpProvisionComputerAccount: LDAP creation failed: 0x6ba
03/01/2017 10:22:56:859 NetpProvisionComputerAccount: Retrying downlevel per options
03/01/2017 10:22:57:015 NetpProvisionComputerAccount: retry status of creating account: 0x0
03/01/2017 10:22:57:015 NetpInitBlobWin7: Constructing blob...
03/01/2017 10:22:57:015 Blob version: 1
03/01/2017 10:22:57:015 	lpDomain: mydomain.local
03/01/2017 10:22:57:015 	lpMachineName: myserver
03/01/2017 10:22:57:015 	lpMachinePassword: <omitted from log>
03/01/2017 10:22:57:015    DomainDnsPolicy:
03/01/2017 10:22:57:015    	Name: MYDOMAIN
03/01/2017 10:22:57:015    	DnsDomainName: mydomain.local
03/01/2017 10:22:57:015    	DnsForestName: mydomain.local
03/01/2017 10:22:57:015    	DomainGuid: 3df83cea-9e20-4bd6-9e2c-deadbeefdead
03/01/2017 10:22:57:015    	Sid: S-1-5-21-839522115-2052111302-9876123412
03/01/2017 10:22:57:015    DcInfo:
03/01/2017 10:22:57:015    	DomainControllerName: \\localdc02.mydomain.local
03/01/2017 10:22:57:015    	DomainControllerAddress: \\10.0.0.2
03/01/2017 10:22:57:015    	DomainControllerAddressType: 1
03/01/2017 10:22:57:015    	DomainGuid: 3df83cea-9e20-4bd6-9e2c-deadbeefdead
03/01/2017 10:22:57:015    	DomainName: mydomain.local
03/01/2017 10:22:57:015    	DnsForestName: mydomain.local
03/01/2017 10:22:57:015    	Flags: 0xe00031fc
03/01/2017 10:22:57:015    	DcSiteName: LU-KIR-01
03/01/2017 10:22:57:015    	ClientSiteName: LU-ARL-01
03/01/2017 10:22:57:015 	Options: 0x40000003
03/01/2017 10:22:57:015 NetpInitBlobWin7: Blob pickling result: 0
03/01/2017 10:22:57:015 ldap_unbind status: 0x0
03/01/2017 10:22:57:015 NetpJoinCreatePackagePart: status:0x0.
03/01/2017 10:22:57:015 NetpAddProvisioningPackagePart: status:0x0.
03/01/2017 10:22:57:015 NetpAddProvisioningPackagePart: status:0x0.
03/01/2017 10:22:57:015 NetpAddProvisioningPackagePart: status:0x0.
03/01/2017 10:22:57:015 NetpAddProvisioningPackagePart: status:0x0.
03/01/2017 10:22:57:015 NetpSerializeProvisioningPackage: status:0x0.
03/01/2017 10:22:57:015 NetpRequestProvisioningPackageInstall:
03/01/2017 10:22:57:015 	cbPackageBinData: 2024
03/01/2017 10:22:57:015 	JoinOptions: 0x27
03/01/2017 10:22:57:015 	ProvisionOptions: 0x40000003
03/01/2017 10:22:57:015 	lpWindowsPath: C:\Windows
03/01/2017 10:22:57:015 NetpRequestProvisioningPackageInstall: Found ODJ blob format: 1.
03/01/2017 10:22:57:015 NetpRequestProvisioningPackageInstall: Found ODJ blob format: 2.
03/01/2017 10:22:57:015 NetpSetPrivileges: Setting backup/restore privileges.
03/01/2017 10:22:57:015 NetpAddPartCollectionToRegistry.
03/01/2017 10:22:57:015 NetpProvGetTargetProductVersion: Target product version: 6.3.9600.16384
03/01/2017 10:22:57:031 NetpAddPartCollectionToRegistry: delete OP state key status: 0x2.
03/01/2017 10:22:57:031 NetpConvertBlobToJoinState: Translating provisioning data to internal format
03/01/2017 10:22:57:031 NetpConvertBlobToJoinState: Selecting version 1
03/01/2017 10:22:57:031 NetpConvertBlobToJoinState: exiting: 0x0
03/01/2017 10:22:57:031 NetpValidateFullJoinState: Validating provisioning data...
03/01/2017 10:22:57:031 NetpValidateFullJoinState: exiting: 0x0
03/01/2017 10:22:57:031 NetpClearFullJoinState:  Removing cached state from the registry...
03/01/2017 10:22:57:031 NetpClearFullJoinState: Status of deleting join state key 0x2
03/01/2017 10:22:57:031 NetpSaveFullJoinStateInternal: Injecting provisioning data into image...
03/01/2017 10:22:57:031 NetpSaveFullJoinStateInternal: exiting: 0x0
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: starting...
03/01/2017 10:22:57:031 	SetHostName:	TRUE
03/01/2017 10:22:57:031 	SetNetBiosName:	TRUE
03/01/2017 10:22:57:031 	SetDnsDomain:	TRUE
03/01/2017 10:22:57:031 	SetCurrentValues:	TRUE
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: using generated Netbiosname:  myserver
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: Setting Hostname to myserver
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: Setting Domain name to mydomain.local
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: Setting NetBios computer name to myserver
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: exiting...
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: starting...
03/01/2017 10:22:57:031 	SetHostName:	FALSE
03/01/2017 10:22:57:031 	SetNetBiosName:	TRUE
03/01/2017 10:22:57:031 	SetDnsDomain:	TRUE
03/01/2017 10:22:57:031 	SetCurrentValues:	TRUE
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: using specified Netbios name: myserver
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: Setting Domain name to mydomain.local
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: Setting NetBios computer name to myserver
03/01/2017 10:22:57:031 NetpSetComputerNamesOffline: exiting...
03/01/2017 10:22:57:031 NetpJoin2RequestPackagePartInstall: Successfully persisted all fields
03/01/2017 10:22:57:031 NetpAddPartCollectionToRegistry: Successfully initiated provisioning package installation: 2/2 part(s) installed.
03/01/2017 10:22:57:031 NetpAddPartCollectionToRegistry: status: 0x0.
03/01/2017 10:22:57:031 NetpOpenRegistry: status: 0x0.
03/01/2017 10:22:57:031 NetpSetPrivileges: status: 0x0.
03/01/2017 10:22:57:031 NetpRequestProvisioningPackageInstall: status: 0x0.
03/01/2017 10:22:57:031 NetpJoinDomainOnDs: Setting netlogon cache.
03/01/2017 10:22:57:047 NetpJoinDomainOnDs: status of setting netlogon cache: 0x0
03/01/2017 10:22:57:047 NetpJoinDomainOnDs: Function exits with status of: 0x0
03/01/2017 10:22:57:047 NetpJoinDomainOnDs: status of disconnecting from '\\localdc02.mydomain.local': 0x0
03/01/2017 10:22:57:047 NetpContinueProvisioningPackageInstall:
03/01/2017 10:22:57:047 	Context: 0
03/01/2017 10:22:57:047 NetpProvGetWindowsImageState: IMAGE_STATE_COMPLETE.
03/01/2017 10:22:57:047 NetpCreatePartListFromRegistry: status: 0x0.
03/01/2017 10:22:57:047 NetpLoadFullJoinState: unable to load NetBiosName (0x0) - ignoring.
03/01/2017 10:22:57:047 NetpLoadFullJoinState: unable to load PersistentSite (0x0) - ignoring.
03/01/2017 10:22:57:047 NetpLoadFullJoinState: unable to load DynamicSite (0x0) - ignoring.
03/01/2017 10:22:57:047 NetpLoadFullJoinState: unable to load PrimaryDNSDomain (0x0) - ignoring.
03/01/2017 10:22:57:047 NetpCompleteOfflineDomainJoin
03/01/2017 10:22:57:047 	fBootTimeCaller: FALSE
03/01/2017 10:22:57:047 	fSetLocalGroups: TRUE
03/01/2017 10:22:57:047 NetpLsaOpenSecret: status: 0xc0000034
03/01/2017 10:22:57:047 NetpGetLsaPrimaryDomain: status: 0x0
03/01/2017 10:22:57:047 NetpJoinDomainLocal: NetpHandleJoinedStateInfo returned: 0x0
03/01/2017 10:22:57:047 NetpLsaOpenSecret: status: 0xc0000034
03/01/2017 10:22:57:094 [000004e8] NetpGetLsaPrimaryDomain: status: 0x0
03/01/2017 10:22:57:109 [000004e8] NetpGetLsaPrimaryDomain: status: 0x0
03/01/2017 10:22:57:109 NetpJoinDomainLocal: NetpManageMachineSecret returned: 0x0.
03/01/2017 10:22:57:109 Calling NetpQueryService to get Netlogon service state.
03/01/2017 10:22:57:109 NetpJoinDomainLocal: NetpQueryService returned: 0x0.
03/01/2017 10:22:57:109 NetpSetLsaPrimaryDomain: for 'MYDOMAIN' status: 0x0
03/01/2017 10:22:57:109 NetpJoinDomainLocal: status of setting LSA pri. domain: 0x0
03/01/2017 10:22:57:109 NetpManageLocalGroupsForJoin: Adding groups for new domain, removing groups from old domain, if any.
03/01/2017 10:22:57:109 NetpManageLocalGroups: Populating list of account SIDs.
03/01/2017 10:22:57:125 NetpManageLocalGroupsForJoin: status of modifying groups related to domain 'MYDOMAIN' to local groups: 0x0
03/01/2017 10:22:57:125 NetpManageLocalGroupsForJoin: INFO: No old domain groups to process.
03/01/2017 10:22:57:125 NetpJoinDomainLocal: Status of managing local groups: 0x0
03/01/2017 10:22:57:125 NetpJoinDomainLocal: status of setting ComputerNamePhysicalDnsDomain to 'mydomain.local': 0x0
03/01/2017 10:22:57:125 NetpJoinDomainLocal: Controlling services and setting service start type.
03/01/2017 10:22:57:125 NetpJoinDomainLocal: Updating W32TimeConfig
03/01/2017 10:22:57:125 NetpUpdateW32timeConfig: 0x0
03/01/2017 10:22:57:125 NetpClearFullJoinState:  Removing cached state from the registry...
03/01/2017 10:22:57:125 NetpClearFullJoinState: Status of deleting join state key 0x0
03/01/2017 10:22:57:125 NetpCompleteOfflineDomainJoin: status: 0x0
03/01/2017 10:22:57:125 NetpJoin2ContinuePackagePartInstall: ignoring Context=0 (work finished already).
03/01/2017 10:22:57:125 NetpContinueProvisioningPackageInstall: Provisioning package installation completed successfully.
03/01/2017 10:22:57:125 NetpContinueProvisioningPackageInstall: delete OP state key status: 0x0.
03/01/2017 10:22:57:125 NetpContinueProvisioningPackageInstall: status: 0xa99.
03/01/2017 10:22:57:125 NetpJoinDomain: NetpCompleteOfflineDomainJoin SUCCESS: Requested a reboot :0x0
03/01/2017 10:22:57:125 NetpDoDomainJoin: status: 0x0
03/01/2017 10:22:59:031 -----------------------------------------------------------------
03/01/2017 10:22:59:031 NetpChangeMachineName: from 'myserver' to 'myserver' using 'mydomain.local\AdminUser' [0x1000]
03/01/2017 10:22:59:031 NetpChangeMachineName: using DnsHostnameToComputerNameEx
03/01/2017 10:22:59:031 NetpChangeMachineName: generated netbios name: 'myserver'
03/01/2017 10:22:59:031 NetpDsGetDcName: trying to find DC in domain 'mydomain.local', flags: 0x1010
03/01/2017 10:23:00:453 NetpDsGetDcName: found DC '\\localdc02.mydomain.local' in the specified domain
03/01/2017 10:23:00:453 NetpGetLsaPrimaryDomain: status: 0x0
03/01/2017 10:23:00:453 NetpGetDnsHostName: Read NV Domain: mydomain.local
03/01/2017 10:23:21:500 NetpGetComputerObjectDn: Unable to bind to DS on '\\localdc02.mydomain.local': 0x6ba
03/01/2017 10:23:21:500 NetpSetDnsHostNameAndSpn: NetpGetComputerObjectDn failed: 0x6ba
03/01/2017 10:23:21:500 ldap_unbind status: 0x0
03/01/2017 10:23:21:500 NetpChangeMachineName: status of setting DnsHostName and SPN: 0x6ba
```

When analyzing netsetup.log, there are two key principles:

- Start from the end, as the error message is likely to be the last attempted action
- Notice that each subpart of the attempt is separated by dashed lines.

In this example, we can see we get a 0x6ba error on the last line. We move backwards until its first apparence and discover that four lines before the end the error first appear with the message ‘Unable to bind the DS’ followed by the name of a domain controller.

When it comes to ‘unable to bind’ messages you can be pretty sure that there are issues at some point to contact the AD DS Service on the mentioned domain controller. You may want to check:

- that the services are running
- that the firewall is properly configured on the domain controller
- that the firewall is properly configured on the member server (there should be no outbound rules, but you may want to check though)
- that there is no other firewall put in place by other teams or that the ports are open.[ PortQry](http://support.microsoft.com/kb/832919) will help you check that. In particular you will need: 
    - UDP/123
    - UDP-TCP/389 and TCP/3268
    - TCP/636 and TCP/3269 if doing LDAPS
    - TCP-UDP/88
    - TCP-UDP/464
    - TCP/135
    - TCP/445
    - TCP/49152-65535 unless you have restricted the dynamic port range. You can check this by issuing ```
         netsh ipv4 show dynamic range tcp
         netsh ipv4 show dynamic range udp
         netsh ipv6 show dynamic range tcp
         netsh ipv6 show dynamic range udp
        ```
- In our case it appears that the first steps are correct because TCP/135 the is open, but the dynamic range is not.