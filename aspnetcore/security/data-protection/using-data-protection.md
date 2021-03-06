---
title: "데이터 보호 Api 시작"
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 39b7a73c-29d4-4137-b311-49529adcf3cb
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/using-data-protection
ms.openlocfilehash: 9489b55b1de626b77bcbe21cb9678e561b559099
ms.sourcegitcommit: 8f4d4fad1ca27adf9e396f5c205c9875a3963664
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/13/2017
---
# <a name="getting-started-with-the-data-protection-apis"></a>데이터 보호 Api 시작

<a name="security-data-protection-getting-started"></a>

가장 간단한 보호 데이터에는 다음 단계로 구성 됩니다.

1. 데이터 보호 공급자에서 데이터 보호자를 만듭니다.

2. 보호 하려는 데이터와 함께 Protect 메서드를 호출 합니다.

3. 다시 일반 텍스트로 변환 하려는 데이터를 사용 하 여 Unprotect 메서드를 호출 합니다.

ASP.NET SignalR 등 대부분의 프레임 워크는 이미 데이터 보호 시스템 구성 및 종속성 주입을 통해 액세스 하는 서비스 컨테이너에 추가 합니다. 다음 예제에서는 종속성 주입을 위한 서비스 컨테이너를 구성 하 고 등록 하는 데이터 보호 스택의 DI 통해 데이터 보호 공급자를 수신, 보호기 및 보호 한 후 보호 해제 데이터 만들기

[!code-csharp[Main](../../security/data-protection/using-data-protection/samples/protectunprotect.cs?highlight=26,34,35,36,37,38,39,40)]

하나 이상을 제공 해야 합니다는 보호기를 만들 때 [목적 문자열](consumer-apis/purpose-strings.md)합니다. "녹색"의 목적은 문자열을 사용 하 여 만든 보호기를 "자주색"의 용도 보호기에서 제공 되는 데이터를 보호할 수 없게 됩니다 예를 들어, 용도 문자열 소비자 간에 격리를 제공 합니다.

>[!TIP]
> IDataProtectionProvider 및 IDataProtector의 인스턴스는 스레드로부터 안전 여러 호출자에 대 한 합니다. 구성 요소 호출 CreateProtector 통해 IDataProtector에 대 한 참조를 가져온 후 해당 참조에 사용할 보호 및 Unprotect를 여러 번 호출 하는 것이 됩니다.
>
>보호 된 페이로드를 확인 하거나 해독할 수 없는 경우 보호 해제에 대 한 호출 CryptographicException 발생 시킵니다. 일부 구성 요소 오류를 무시 하려는 경우가 있을 수 하는 동안 작업을 보호 해제 인증 쿠키를 읽는 구성 요소 수 있습니다이 오류를 처리 하 고 전혀 쿠키가 없는 이전의 요청 처럼 처리할 대신 완전 한 요청에 실패 합니다. 구체적으로이 동작을 방지 하는 구성 요소는 모든 예외를 받아들이지 않고 CryptographicException을 catch 해야 합니다.
