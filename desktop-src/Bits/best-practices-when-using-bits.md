---
title: Best Practices When Using BITS
description: This section contains information you should consider when designing an application that uses BITS.
ms.assetid: f4a09a80-2a85-4b59-b0fd-c23c128973f7
ms.technology: desktop
ms.prod: windows
ms.author: windowssdkdev
ms.topic: article
ms.date: 05/31/2018
---

# Best Practices When Using BITS

This section contains information you should consider when designing an application that uses BITS.

## User context or service context

BITS transfers files only when the job's owner is logged on to the computer (the user must have logged on interactively). BITS does not support the **RunAs** command. For more details, see [Users and Network Connections](users-and-network-connections.md).

If you do not require a user's context for your application, consider writing a service running as LocalSystem, LocalService, or NetworkService instead. These system accounts are always logged on, so the transfer is not subject to a user logging off. However, if you impersonate a user when you create the job, the interactive logon rules apply. For more details, see [Service Accounts and BITS](service-accounts-and-bits.md).

## Setting credentials for proxy and server authentication

If you expect the proxy or server to require user credentials, you must provide the credentials to BITS. To specify the credentials, call the [**IBackgroundCopyJob2::SetCredentials**](/windows/desktop/api/Bits1_5/nf-bits1_5-ibackgroundcopyjob2-setcredentials) method. BITS supports Basic, Digest, Negotiate, NTLM, and Passport authentication schemes.

For details on authentication, see [Authentication](authentication.md).

## Specifying proxy settings for user accounts and service accounts

By default, BITS uses the user's Internet Explorer proxy settings. To override the user's Internet Explorer proxy settings, call the [**IBackgroundCopyJob::SetProxySettings**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-setproxysettings) method.

The Internet Explorer proxy settings do not apply to system accounts. If your application is a service running as LocalSystem, LocalService, or NetworkService, you must call [**IBackgroundCopyJob::SetProxySettings**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-setproxysettings) to specify the proxy settings. As an alternative, you can use the **/Util /SetIEProxy** switches of BitsAdmin.exe to set Internet Explorer proxy settings for the LocalSystem, LocalService, or NetworkService system account. For details, see [BitsAdmin Tool](bitsadmin-tool.md).

BITS does not recognize the proxy settings that are set using the Proxycfg.exe file.

## Specifying user-specific settings for authenticating proxies

If you are using BITS in an environment that requires proxy authentication while running as an account without usable NTLM or Kerberos credentials in the machine's network domain, you must take extra steps to authenticate properly by using the credentials of another user account that does have credentials on the domain. This is a typical scenario when your BITS code is running as a system service such as LocalService, NetworkService, or LocalSystem, as those accounts do not have usable NTLM or Kerberos credentials.

For details on how authentication works in this scenario, see [Authentication.](https://www.bing.com/search?q=Authentication.)

## Scalability

If more than 100 jobs are in the queue, performance may start to decrease depending on the composition of the job. BITS uses the [MaxJobsPerMachine](group-policies.md) policy setting to impose a hard limit on the number of jobs in the queue. Applications should limit the number of their jobs to about 10, so that multiple applications will have less of a chance of exceeding the 100-job guideline. Typically, an application with a large number of jobs to submit would first submit 10 jobs and then submit one at a time as each job finishes.

The number of files in the job should also be limited to a maximum of 10 files. If you want to transfer a large number of files for a job, consider creating a CAB file that contains all the files instead.

## Jobs are persistent

Jobs remain in the queue until you call the [**IBackgroundCopyJob::Complete**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-complete) or [**IBackgroundCopyJob::Cancel**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-cancel) method. The files in the job are not available to the user until you call **Complete**. Typically, you call **Complete** when the state of the job is **BG\_JOB\_STATE\_TRANSFERRED**, and you call **Cancel** when the job is in the **BG\_JOB\_STATE\_TRANSIENT\_ERROR** or **BG\_JOB\_STATE\_ERROR** state and can no longer make progress.

If you do not call the [**Complete**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-complete) method or the [**Cancel**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-cancel) method within 90 days (default [JobInactivityTimeout](group-policies.md) Group Policy), the service cancels the job. You should always call the **Complete** or the **Cancel** method and not rely on the JobInactivityTimeout policy to cleanup your jobs. Jobs left in the queue may prevent users from creating other jobs if the MaxJobsPerUser or MaxJobsPerMachine policy limit is reached.

## When to use foreground or background priority

Unless the job is time critical or the user is actively waiting, you should always use a background priority. However, there are times when you may want to switch from background priority to foreground priority, for example, when the proxy or server does not support the Content-Range header, or antivirus software on the client removes the range header request. Switching to foreground priority works only for those files whose file size is less than 2 GB. For an example, see the implementation for the [**IBackgroundCopyCallback::JobError**](/windows/desktop/api/Bits/nn-bits-ibackgroundcopycallback) method. Also note that if the foreground job is then interrupted due to a network disconnect or the user logging off, the job will fail because BITS will send a range request to try to restart the transfer from where it left off.

For information on the available priorities and how BITS uses the priority level to schedule jobs, see [**BG\_JOB\_PRIORITY**](/windows/desktop/api/Bits/ne-bits-__midl_ibackgroundcopyjob_0001).

## Transient and fatal errors

Some errors are recoverable and some are not. For example, the error "Server is Unavailable" is a recoverable error, and the error "Access Denied" is a fatal error. BITS puts recoverable errors in a transient error state and tries the job again after a specified interval. If the job is unable to make progress, BITS moves the job to a fatal error state. Use the [**IBackgroundCopyJob::SetMinimumRetryDelay**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-setminimumretrydelay) and [**IBackgroundCopyJob::SetNoProgressTimeout**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-setnoprogresstimeout) methods to control how BITS processes transient errors.

For foreground jobs, you should limit the amount of time you let a job stay in the transient error state and try to recover. Use the [**SetNoProgressTimeout**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-setnoprogresstimeout) method to limit the amount of time a job stays in the transient error state or to force the job into the fatal error state. If you let the job try to recover, you should use the [**SetMinimumRetryDelay**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-setminimumretrydelay) method to set the minimum retry delay to 60 seconds or call the [**IBackgroundCopyJob::Resume**](/windows/desktop/api/Bits/nf-bits-ibackgroundcopyjob-resume) method to activate the job again.

For more information, see [**BG\_JOB\_STATE**](/windows/desktop/api/Bits/ne-bits-__midl_ibackgroundcopyjob_0002), [Life Cycle of a BITS Job](life-cycle-of-a-bits-job.md), and [Handling Errors](handling-errors.md).

## Measuring network bandwidth usage

BITS uses the client's network adapter to measure available network bandwidth. Because BITS is not able to measure bandwidth beyond the client, BITS may congest the WAN link. To reduce congestion on the WAN link, you can use the **MaxInternetBandwidth** group policy to limit the amount of bandwidth that the client uses. For more information, see [Network Bandwidth](network-bandwidth.md) and [Group Policies](group-policies.md).

If you are writing an application that many clients will use to download files from a given server, you should consider a scheme that staggers the download requests so you do not overload the server with requests.

## HTTP Headers can be in any case

The HTTP standards have always said that HTTP headers must be treated as case-insensitive (RFC 7230 section 3.2). The most recent HTTP standard, RFC 7540, goes further and says that HTTP/2 traffic must compare the headers as case-insensitive and must present headers in lower case (RFC 6540, section 8.1.2). Even when traffic is sent with non lower-case headers, proxies may well choose to force the headers to lower-case.

 

 



