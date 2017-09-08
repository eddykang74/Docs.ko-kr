---
title: "ASP.NET Core 응용 프로그램에서 열려 리디렉션 공격 방지 | Microsoft Docs"
author: ardalis
description: "ASP.NET Core 응용 프로그램에 대 한 열기 리디렉션 공격을 방지 하는 방법을 보여 줍니다."
keywords: "ASP.NET Core, 보안, 열기 리디렉션 공격"
ms.author: riande
manager: wpickett
ms.date: 07/7/2017
ms.topic: article
ms.assetid: 4604e563-e91a-4ecd-b7ed-00b3f1eee2b5
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/preventing-open-redirects
ms.openlocfilehash: 7a62b08d641de5a9df5c2f7d89bf6bf97ed8e39d
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/11/2017
---
# <a name="preventing-open-redirect-attacks-in-an-aspnet-core-app"></a><span data-ttu-id="e2c32-104">ASP.NET Core 응용 프로그램에서 열려 리디렉션 공격 방지</span><span class="sxs-lookup"><span data-stu-id="e2c32-104">Preventing Open Redirect Attacks in an ASP.NET Core app</span></span>

<span data-ttu-id="e2c32-105">잠재적으로 쿼리 문자열 또는 폼 데이터와 같은 요청을 통해 지정 된 URL로 리디렉션하는 웹 응용 프로그램 외부, 악성 URL로 사용자를 리디렉션하기와 훼손 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-105">A web app that redirects to a URL that is specified via the request such as the querystring or form data can potentially be tampered with to redirect users to an external, malicious URL.</span></span> <span data-ttu-id="e2c32-106">이 변조 열려 리디렉션 공격을 이라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-106">This tampering is called an open redirection attack.</span></span>

<span data-ttu-id="e2c32-107">응용 프로그램 논리를 지정 된 URL로 리디렉션합니다, 때마다와 리디렉션 URL이 손상 되지 않았음을 확인 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-107">Whenever your application logic redirects to a specified URL, you must verify that the redirection URL hasn't been tampered with.</span></span> <span data-ttu-id="e2c32-108">ASP.NET Core 응용 프로그램 열기 (라고도 열려 리디렉션) 리디렉션 공격 으로부터 보호 하려면 기본 제공 기능이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-108">ASP.NET Core has built-in functionality to help protect apps from open redirect (also known as open redirection) attacks.</span></span>

## <a name="what-is-an-open-redirect-attack"></a><span data-ttu-id="e2c32-109">열린 리디렉션 공격 무엇입니까?</span><span class="sxs-lookup"><span data-stu-id="e2c32-109">What is an open redirect attack?</span></span>

<span data-ttu-id="e2c32-110">자주 웹 응용 프로그램 인증을 요구 하는 리소스에 액세스할 때 사용자가 로그인 페이지에 게 리디렉션합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-110">Web applications frequently redirect users to a login page when they access resources that require authentication.</span></span> <span data-ttu-id="e2c32-111">리디렉션 typlically 포함 한 `returnUrl` querystring 매개 변수 사용자 성공적으로 로그인 한 후 원래 요청 된 url 반환 될 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-111">The redirection typlically includes a `returnUrl` querystring parameter so that the user can be returned to the originally requested URL after they have successfully logged in.</span></span> <span data-ttu-id="e2c32-112">사용자가 인증 한 후에 원래 요청 했던 URL로 리디렉션됩니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-112">After the user authenticates, they are redirected to the URL they had originally requested.</span></span>

<span data-ttu-id="e2c32-113">대상 URL 요청 쿼리 문자열에 지정 되어 있으므로 악의적인 사용자는 쿼리 문자열 변조할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-113">Because the destination URL is specified in the querystring of the request, a malicious user could tamper with the querystring.</span></span> <span data-ttu-id="e2c32-114">변조 된 querystring 사이트 사용자를 외부, 악성 사이트로 리디렉션할를 허용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-114">A tampered querystring could allow the site to redirect the user to an external, malicious site.</span></span> <span data-ttu-id="e2c32-115">이 기술은 열려 리디렉션 (또는 리디렉션) 공격을 이라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-115">This technique is called an open redirect (or redirection) attack.</span></span>

### <a name="an-example-attack"></a><span data-ttu-id="e2c32-116">예제 공격</span><span class="sxs-lookup"><span data-stu-id="e2c32-116">An example attack</span></span>

<span data-ttu-id="e2c32-117">악의적인 사용자를 사용자의 자격 증명 또는 응용 프로그램에서 중요 한 정보를 악의적인 사용자가 액세스를 허용 하는 방식의 공격을 개발할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-117">A malicious user could develop an attack intended to allow the malicious user access to a user's credentials or sensitive information on your app.</span></span> <span data-ttu-id="e2c32-118">포함 된 사이트의 로그인 페이지에 대 한 링크를 클릭 하 여 사용자를 유도 방지할 수 있었던 공격을 시작 하려면 한 `returnUrl` URL에 추가 하는 쿼리 문자열 값입니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-118">To begin the attack, they convince the user to click a link to your site's login page, with a `returnUrl` querystring value added to the URL.</span></span> <span data-ttu-id="e2c32-119">예를 들어는 [NerdDinner.com](http://nerddinner.com) (ASP.NET MVC 용으로 작성 된) 샘플 응용 프로그램에는 로그인 페이지가 여기에 포함 되어: ``http://nerddinner.com/Account/LogOn?returnUrl=/Home/About``합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-119">For example, the [NerdDinner.com](http://nerddinner.com) sample application (written for ASP.NET MVC) includes such a login page here: ``http://nerddinner.com/Account/LogOn?returnUrl=/Home/About``.</span></span> <span data-ttu-id="e2c32-120">방지할 수 있었던 공격 후 다음이 단계를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-120">The attack then follows these steps:</span></span>

1. <span data-ttu-id="e2c32-121">사용자는 링크를 클릭 ``http://nerddinner.com/Account/LogOn?returnUrl=http://nerddiner.com/Account/LogOn`` (두 번째 URL은 nerddi**n**어, 하지 nerddi**nn**어).</span><span class="sxs-lookup"><span data-stu-id="e2c32-121">User clicks a link to ``http://nerddinner.com/Account/LogOn?returnUrl=http://nerddiner.com/Account/LogOn`` (note, second URL is nerddi**n**er, not nerddi**nn**er).</span></span>
2. <span data-ttu-id="e2c32-122">사용자가 성공적으로 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-122">The user logs in successfully.</span></span>
3. <span data-ttu-id="e2c32-123">사용자가을 (사이트)에 의해 리디렉션되 ``http://nerddiner.com/Account/LogOn`` (실제 사이트 처럼 보이는 악성 사이트).</span><span class="sxs-lookup"><span data-stu-id="e2c32-123">The user is redirected (by the site) to ``http://nerddiner.com/Account/LogOn`` (malicious site that looks like real site).</span></span>
4. <span data-ttu-id="e2c32-124">사용자가 다시 로그인 (자격 증명 사이트 악의적인 제공)을 실제 사이트로 다시 리디렉션됩니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-124">The user logs in again (giving malicious site their credentials) and is redirected back to the real site.</span></span>

<span data-ttu-id="e2c32-125">사용자는 가능성이 생각 로그인의 첫 번째 시도가 실패와 두 번째 식의 성공 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-125">The user will likely believe their first attempt to log in failed, and their second one was successful.</span></span> <span data-ttu-id="e2c32-126">됩니다 가능성이 인식 하지 않는 유지 자격 증명 노출 되었다고 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-126">They'll most likely remain unaware their credentials have been compromised.</span></span>

![열기 리디렉션 공격 프로세스](preventing-open-redirects/_static/open-redirection-attack-process.png)

<span data-ttu-id="e2c32-128">로그인 페이지 외에도 일부 사이트 리디렉션 페이지 또는 끝점을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-128">In addition to login pages, some sites provide redirect pages or endpoints.</span></span> <span data-ttu-id="e2c32-129">응용 프로그램에 열려 리디렉션 있는 페이지를 가정해 보십시오. ``/Home/Redirect``합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-129">Imagine your app has a page with an open redirect, ``/Home/Redirect``.</span></span> <span data-ttu-id="e2c32-130">공격자가 만들 수, 예를 들어,로 이동 하는 전자 메일의 링크 ``[yoursite]/Home/Redirect?url=http://phishingsite.com/Home/Login``합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-130">An attacker could create, for example, a link in an email that goes to ``[yoursite]/Home/Redirect?url=http://phishingsite.com/Home/Login``.</span></span> <span data-ttu-id="e2c32-131">일반적인 사용자 URL 확인 되며 참조 사이트 이름으로 시작 됩니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-131">A typical user will look at the URL and see it begins with your site name.</span></span> <span data-ttu-id="e2c32-132">신뢰 하는, 해당 링크를 클릭 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-132">Trusting that, they will click the link.</span></span> <span data-ttu-id="e2c32-133">열기 리디렉션은 다음 사용자에 게 보낼 피싱 사이트로 실제로 사용자 작업과 동일 하 고 사용자 했을 것 생각 하는 로그인이 사이트입니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-133">The open redirect would then send the user to the phishing site, which looks identical to yours, and the user would likely login to what they believe is your site.</span></span>

## <a name="protecting-against-open-redirect-attacks"></a><span data-ttu-id="e2c32-134">열린 리디렉션 공격 으로부터 보호</span><span class="sxs-lookup"><span data-stu-id="e2c32-134">Protecting against open redirect attacks</span></span>

<span data-ttu-id="e2c32-135">웹 응용 프로그램을 개발할 때는으로 신뢰할 수 없는 모든 사용자가 제공한 데이터를 처리 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-135">When developing web applications, treat all user-provided data as untrustworthy.</span></span> <span data-ttu-id="e2c32-136">리디렉션하는 URL의 내용에 따라 사용자 기능에이 응용 프로그램에 앱 내에서 (또는 알려진된 URL, 쿼리 문자열에서 제공 될 수 있습니다 내보내지 URL) 그러한 리디렉션을 로컬로 수행만 됩니다를 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-136">If your application has functionality that redirects the user based on the contents of the URL,  ensure that such redirects are only done locally within your app (or to a known URL, not any URL that may be supplied in the querystring).</span></span>

### <a name="localredirect"></a><span data-ttu-id="e2c32-137">LocalRedirect</span><span class="sxs-lookup"><span data-stu-id="e2c32-137">LocalRedirect</span></span>

<span data-ttu-id="e2c32-138">사용 하 여는 ``LocalRedirect`` 도우미 메서드는 기본에서 `Controller` 클래스:</span><span class="sxs-lookup"><span data-stu-id="e2c32-138">Use the ``LocalRedirect`` helper method from the base `Controller` class:</span></span>

```
public IActionResult SomeAction(string redirectUrl)
{
    return LocalRedirect(redirectUrl);
}
```

<span data-ttu-id="e2c32-139">``LocalRedirect``로컬이 아닌 URL을 지정 하는 경우 예외가 throw 됩니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-139">``LocalRedirect`` will throw an exception if a non-local URL is specified.</span></span> <span data-ttu-id="e2c32-140">그렇지 않은 경우와 동일 하 게 작동는 ``Redirect`` 메서드.</span><span class="sxs-lookup"><span data-stu-id="e2c32-140">Otherwise, it behaves just like the ``Redirect`` method.</span></span>

### <a name="islocalurl"></a><span data-ttu-id="e2c32-141">IsLocalUrl</span><span class="sxs-lookup"><span data-stu-id="e2c32-141">IsLocalUrl</span></span>

<span data-ttu-id="e2c32-142">사용 하 여는 [IsLocalUrl](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.iurlhelper#Microsoft_AspNetCore_Mvc_IUrlHelper_IsLocalUrl_System_String_) 리디렉션하기 전에 Url을 테스트 하려면:</span><span class="sxs-lookup"><span data-stu-id="e2c32-142">Use the [IsLocalUrl](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.iurlhelper#Microsoft_AspNetCore_Mvc_IUrlHelper_IsLocalUrl_System_String_) method to test URLs before redirecting:</span></span>

<span data-ttu-id="e2c32-143">다음 예제에서는 URL을 리디렉션하기 전에 로컬 인지 여부를 확인 하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-143">The following example shows how to check whether a URL is local before redirecting.</span></span>

```
private IActionResult RedirectToLocal(string returnUrl)
{
    if (Url.IsLocalUrl(returnUrl))
    {
        return Redirect(returnUrl);
    }
    else
    {
        return RedirectToAction(nameof(HomeController.Index), "Home");
    }
}
```

<span data-ttu-id="e2c32-144">`IsLocalUrl` 메서드 실수로 악성 사이트에 리디렉션되지 사용자가 보호 합니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-144">The `IsLocalUrl` method protects users from being inadvertently redirected to a malicious site.</span></span> <span data-ttu-id="e2c32-145">로컬이 아닌 URL을 로컬 URL 필요한 상황에 제공 되는 제공 된 URL의 세부 정보를 로깅할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-145">You can log the details of the URL that was provided when a non-local URL is supplied in a situation where you expected a local URL.</span></span> <span data-ttu-id="e2c32-146">로깅 리디렉션 Url 리디렉션 공격 진단에 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e2c32-146">Logging redirect URLs may help in diagnosing redirection attacks.</span></span>