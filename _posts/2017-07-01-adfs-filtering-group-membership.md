---
id: 732
title: 'Filtering group membership'
date: '2017-07-01T15:48:49+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=732'
permalink: /2017/07/01/adfs-filtering-group-membership/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"733";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/05/ADFS-Windows-Account-Name-Rule-Properties.png
categories:
    - 'ADFS-AD Federation Services'
tags:
    - 'ADFS language'
    - claim
    - filter
    - 'group membership'
    - issue
    - security
    - token
    - tutorial
---

When you’re establishing a relying party trust with a provider filtering group membership you send through your AD FS Farm is often a prerequisite, either for performance issues -so that the token is not too big- or for security reasons as you do not want your provider to know your organisation, in particular when Windows groups are used as a RBAC model.

Unfortunately, by default the wizard only proposes to

- send all the group memberships
- do a 1:1 filter or search and replace on a specified group.

When you have design your application so every role is a Windows group membership and your applications has multiple roles, the task can be tedious in a scenario where “Your App” may have dozen of roles and each user may have read, write, or whatever additional permission.

Fortunately, the wizard is just a prebuilt set of rules around the ADFS language to create and issue tokens. We can therefore build our own rules, and this will serve us as a quick introduction to the ADFS language.

To have some boiler plate examples of the ADFS language, you can always select the ‘View language’ button at the bottom left of pre-built rule:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/05/ADFS-Windows-Account-Name-Rule-Properties.png)](https://dimitri.janczak.net/wp-content/uploads/2017/05/ADFS-Windows-Account-Name-Rule-Properties.png)In this case, we can such that such a rule is equivalent to a custom rule whose contents would be:

```
<pre class="lang:default decode:true" title="Windows Account Name claim">c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
 => issue(claim = c);
```

In this example the variable is named “c\* and if it matches the type windowsaccountname, it is simply issued. On a general note, an ADFS rule is “Condition =&gt; Action”. If condition is empty, the action is always done.

The action can be:

- issue to issue the claim
- add to add a piece of claim to a store to issue later as you cannot “remove” something from a claim.

In our example to filter the group membership, we will:

1. store all the claims corresponding to the group membership
2. issue a claim if it is a group and starts with the name we want as we have decided that our RBAC model for the application make the group in form of ‘MyApp\_XXX’

The first rule is then written

```
<pre class="lang:default decode:true" title="5.1 Get Group Membership from LDAP Claims without domain name">c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
=> add(store = "Active Directory", types = ("http://schemas.xmlsoap.org/claims/Group"), query = ";memberOf;{0}", param = c.Value);
```

Note the verb ‘add’ to add the data to our temporary claim store.

The second rule can be expressed as:

```
<pre class="lang:default decode:true " title="5.2 Filter group membership starting with MyApp_">c:[Type == "http://schemas.xmlsoap.org/claims/Group", Value =~ "^(?i)MyApp_"]
=> issue(claim = c);
```

Please note that the rule order is important, since we cannot change the store once the claim is issued

Needless to say that those 2 rules are managing the group membership part of the claims you’re going to send to your relying party. In most cases you may want to send other claims. A standard list could be:

1. Windows Account Name (standard ADFS Rule)
2. Name (standard ADFS Rule)
3. Get Group Membership from LDAP Claims without domain name (Custom Rule)
4. Filter group membership starting with MyApp\_ (custom rule)