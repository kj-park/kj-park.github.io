---
author: "kj-park"
layout: "posts"
---
# How to restore deleted user accounts and their group memberships in Active Directory using the Authoritative restore method

This article is based on the Microsoft KB840001 and my test lab.

<table class="Reference">
    <tr><td>Article ID</td><td>840001</td></tr>
    <tr><td>Title</td><td>How to restore deleted user accounts and their group memberships in Active Directory</td></tr>
    <tr><td>Link</td><td><a href="https://support.microsoft.com/en-us/kb/840001">https://support.microsoft.com/en-us/kb/840001</a></td></tr>
</table>

## [Environment]

- A Single Forest and A Single Domain Active Directory Model.
- Domain Name: test.lab
- Windows Server 2003 SP1 based Domain Controllers in Active directory.
- Two domain controller in Active Directory and the same AD site. 
    - DC01: DC role with PDC and GC
    - DC02: DC role


## [Scenario]

- TimeStamp A: 
    - test01 ~ test10 users in "UserAndGroups" OU
    - testgroup01 ~ testgroup02 groups in "UserAndGroups" OU
    - testgroup01's member: test01 ~ test10
    - testgroup02's member: test04 ~ test07

- TimeStamp B: 
    - Backup the System State for DC01

- TimeStamp C: 
    - Create test11 ~ test20 users in "UserAndGroups" OU
    - Create testgroup03 ~ testgroup04 groups in "UserAndGroups" OU
    - Add member to testgroup01: test11 ~ test20
    - testgroup03's member: test08 ~ test18
    - testgroup04's member: test11 ~ test20

- TimeStamp D: 
    - Delete some users and groups accidentally
    - Occurred the Replication of these deletions to All DCs
    - Deleted Objects: test08 ~ test12, testgroup02 ~ testgroup03

- TimeStamp E: 
    - Disable the Inbound Replication for All DCs
        ```batch
        repadmin /options <dc_name> +DISABLE_INBOUND_REPL
        ```

- TimeStamp F: 
    - Backup the System State for DC01 for roll-back to the Current State

- TimeStamp G: 
    - Reset the password for DSRM local administrator.
        ```batch
        NTDSUTIL "Set DSRM Password" "Reset Password on server UMTDC" Quit Quit
        ```

- TimeStamp H: 
    - Restore the System State for DC01 with making sure that you restore junction points in advanced option

- TimeStamp I: 
    - Restart DC01 in Dsrepair mode and logon as DSRM local administrator

- TimeStamp J: 
    - Auth Restore the deleted user and group accounts using ntdsutil tool
        ```batch
        NTDSUTIL "Authoritative restore" "Restore subtree OU=UserAndGroups,DC=test,DC=lab" Quit Quit
        ```

- TimeStamp K: 
    - disconnect the DC01's all network cable and restart in normal Active Directory Mode

- TimeStamp L: 
    - Disable the Inbound Replication for DC01
        ```batch
        repadmin /options DC01 +DISABLE_INBOUND_REPL
        ```

- TimeStamp M: 
    - Outbound-replicate the auth-restored object to DC02 and sync all between DCs
        ```batch
        repadmin /options DC02 -DISABLE_INBOUND_REPLrepadmin /syncall /d /e /P DC01 DC=test,DC=lab
        repadmin /options DC01 -DISABLE_INBOUND_REPL
        repadmin /syncall DC01 DC=test,DC=labrepadmin /syncall DC02 DC=test,DC=lab
        ```

## [Result]

In  "UserAndGroups" OU, the follow objects are restored or existed

Users: test01 ~ test10, test13 ~ test20 Groups: testgroup01, testgroup02, testgroup04

As result, cannot restore the deleted objects which are created after backup.
