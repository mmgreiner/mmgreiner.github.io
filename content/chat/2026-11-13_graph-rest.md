---
title: 🤖 Microsoft Graph Authorization
description: Microsoft Graph enforces authorization using the access token presented
  with each request. This document explains the intersection of app permissions and
  user rights for SharePoint documents, and provides guidance on delegating access
  and limiting app-only permissions.
categories:
- authoring
tags:
- Microsoft Graph
- authorization
- SharePoint
- api documentation
- security
date: 2026-11-13
draft: false
showToc: false
---


## How does Microsoft Graph use authorization?

Microsoft Graph enforces authorization using the access token presented with each request. For SharePoint documents, access is generally the intersection of **the app’s granted Graph permissions** and, in delegated access, **the signed-in user’s SharePoint rights**. <citation src="1"></citation>

- **Delegated access:** The app acts for a signed-in user. Even if the app has a broad delegated scope such as `Files.Read.All`, it can only retrieve files that user can access in SharePoint—through direct sharing, group membership, ownership, or other supported grants.
- **Application (app-only) access:** The app acts as itself, without a signed-in user. Its access is determined by its application permissions and any resource-level restrictions; broad permissions can allow access across the organization. <citation src="1"></citation>
- **Narrowing app-only access:** For a service that only needs particular SharePoint sites or files, consider selected permissions such as `Sites.Selected` or `Files.SelectedOperations.Selected`, with access separately granted to the specific site or files. <citation src="2"></citation>

So, if your app retrieves a document and then serves it to someone else, Graph’s check applies to the identity in the token—not automatically to that later recipient. Your app should avoid exposing the content to people who aren’t entitled to it, and should use delegated access or carefully restricted app-only permissions as appropriate. <citation src="1,2"></citation>



## How do you keep authorization with the document?

A downloaded copy **doesn’t automatically keep SharePoint’s access controls**. Graph authorizes the download when your app calls it; once the bytes are copied elsewhere, SharePoint permissions don’t travel with the file. You need to preserve authorization in the way you deliver or store the content.

**Best option: keep the document in SharePoint and serve it on demand.** Store the SharePoint `driveId`/`itemId` (and perhaps a link), not a permanent copy. When a user requests it, authenticate that user and use a **delegated Graph token** to fetch or stream the file. Graph then evaluates both the app’s delegated scopes and the user’s SharePoint access; the user can’t gain more access through the app than either permits. <citation src="2"></citation>

If you must cache or copy documents into your own system, treat that system as a separate security boundary:

- Authenticate each viewer and enforce an authorization policy before returning content.
- Keep the copy’s ACLs synchronized with SharePoint changes—or re-check access against SharePoint at request time. Account for revoked access, changed group membership, inheritance, sharing links, and deletions.
- Don’t expose a shared app-only download endpoint that returns a document based only on the app’s broad permissions.
- Use least-privilege Graph permissions. For app-only access, selected scopes can restrict the app to specific sites, libraries, folders, or files, but they constrain **the app**, not each person using your system. <citation src="2"></citation>

If you mean the downloaded file itself must remain protected when it leaves SharePoint, investigate **Microsoft Purview Information Protection sensitivity labels and encryption**. That’s separate from Graph’s authorization checks and needs to be configured for your organization and file types.

In short: use delegated, per-user retrieval when you need SharePoint’s live permissions to govern access; for stored copies, implement and maintain your own authorization controls or apply persistent protection. <citation src="1,2"></citation>


🤖 created with help of ai
