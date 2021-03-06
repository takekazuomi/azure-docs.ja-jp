---
title: 要求の制限 - Translator Text API
titleSuffix: Azure Cognitive Services
description: この記事では、Translator Text API に対する要求の制限を示します。 料金は、要求ごとに 5,000 文字に制限された要求の頻度ではなく、文字数に基づいて発生します。 文字の制限はサブスクリプションに基づき、F0 では 1 時間あたり 200 万文字に制限されます。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 11/15/2018
ms.author: erhopf
ms.openlocfilehash: f89e7e674efe3a823b7c969840772565650d8d07
ms.sourcegitcommit: 90cec6cccf303ad4767a343ce00befba020a10f6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/07/2019
ms.locfileid: "55859472"
---
# <a name="request-limits-for-translator-text"></a>Translator Text に対する要求の制限

この記事では、Translator Text API に対する調整の制限を示します。 サービスには、翻訳、音訳、文の長さの検出、言語の検出、および代替の翻訳が含まれます。

## <a name="character-limits-per-request"></a>要求あたりの文字制限

各要求は 5,000 文字に制限されています。 要求の数ではなく、文字単位で課金されます。 より短い要求を送信し、常にいくつかの要求を未処理にすることをお勧めします。

Translator Text API に対する未処理の要求の数に制限はありません。

## <a name="character-limits-per-hour"></a>時間あたりの文字制限

時間あたりの文字制限は、Translator Text のサブスクリプション レベルに基づきます。 これらの制限に達するか、またはをそれを超えた場合、クォータ不足の応答を受け取る可能性があります。

| レベル | 文字数制限 |
|------|-----------------|
| F0 | 200 万文字/時間 |
| S1 | 4,000 万文字/時間 |
| S2 | 4,000 万文字/時間 |
| S3 | 12,000 万文字/時間 |
| S4 | 20,000 万文字/時間 |

これらの制限は、Microsoft の一般的なシステムに適用されます。 Microsoft の Translator Hub を使用するカスタム翻訳システムは、1 秒あたり 1,800 文字に制限されます。

## <a name="latency"></a>Latency

Translator Text の最大待機時間は 13 秒です。 この時間までに、結果またはタイムアウト応答を受け取ります。 通常、応答は 150 ミリ秒から 300 ミリ秒で返されます。 応答時間は、サイズまたは要求と言語の組み合わせによって異なります。

## <a name="sentence-length-limits"></a>文の長さの制限

[BreakSentence](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-break-sentence) 関数を使用する場合、文の長さは 275 文字に制限されます。 以下の言語については例外があります。

| 言語 | コード | 文字数制限 |
|----------|------|-----------------|
| 中国語 | zh | 132 |
| ドイツ語 | de | 290 |
| イタリア語 | it | 280 |
| 日本語 | ja | 150 |
| ポルトガル語 | pt | 290 |
| スペイン語 | es | 280 |
| イタリア語 | it | 280 |
| タイ語 | th | 258 |

> [!NOTE]
> この制限は、翻訳には適用されません。

## <a name="next-steps"></a>次の手順

* [料金](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)
* [リージョン別の提供状況](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services)
* [v3 Translator Text API のリファレンス](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
