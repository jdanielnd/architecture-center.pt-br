---
title: Escolhendo uma tecnologia de processamento de idioma natural
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 20dc51e661befcc09dd1751b031d445ff2b9fa1a
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486481"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="d19c0-102">Escolhendo uma tecnologia de processamento de idioma natural no Azure</span><span class="sxs-lookup"><span data-stu-id="d19c0-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="d19c0-103">O processamento de texto de forma livre é feito em documentos que contêm parágrafos de texto, normalmente, para fins de pesquisa de apoio, mas também é usado para executar outras tarefas de NLP (processamento de idioma natural) como análise de sentimento, detecção de tópico, detecção de idioma, extração de frases-chave e categorização de documento.</span><span class="sxs-lookup"><span data-stu-id="d19c0-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="d19c0-104">Este artigo se concentra nas opções de tecnologia que servem para dar suporte às tarefas de NLP.</span><span class="sxs-lookup"><span data-stu-id="d19c0-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="d19c0-105">Quais são as opções disponíveis ao escolher um serviço NLP?</span><span class="sxs-lookup"><span data-stu-id="d19c0-105">What are your options when choosing an NLP service?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="d19c0-106">No Azure, os seguintes serviços fornecem funcionalidades de NLP (processamento de idioma natural):</span><span class="sxs-lookup"><span data-stu-id="d19c0-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="d19c0-107">Azure HDInsight com Spark e Spark MLlib</span><span class="sxs-lookup"><span data-stu-id="d19c0-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="d19c0-108">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="d19c0-108">Azure Databricks</span></span>](/azure/azure-databricks/what-is-azure-databricks)
- [<span data-ttu-id="d19c0-109">Serviços Cognitivos da Microsoft</span><span class="sxs-lookup"><span data-stu-id="d19c0-109">Microsoft Cognitive Services</span></span>](/azure/cognitive-services/welcome)

## <a name="key-selection-criteria"></a><span data-ttu-id="d19c0-110">Principais critérios de seleção</span><span class="sxs-lookup"><span data-stu-id="d19c0-110">Key selection criteria</span></span>

<span data-ttu-id="d19c0-111">Para restringir as opções, comece respondendo a estas perguntas:</span><span class="sxs-lookup"><span data-stu-id="d19c0-111">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="d19c0-112">Você deseja usar modelos predefinidos?</span><span class="sxs-lookup"><span data-stu-id="d19c0-112">Do you want to use prebuilt models?</span></span> <span data-ttu-id="d19c0-113">Em caso afirmativo, considere o uso das APIs oferecidas pelos Serviços Cognitivos da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="d19c0-113">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="d19c0-114">Você precisa treinar modelos personalizados em um corpo grande de dados de texto?</span><span class="sxs-lookup"><span data-stu-id="d19c0-114">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="d19c0-115">Em caso afirmativo, considere o uso do Azure HDInsight com Spark MLlib e Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="d19c0-115">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="d19c0-116">Você precisa de funcionalidades de NLP de baixo nível como geração de tokens, lematização e TF/IDF (frequência de termo/frequência de documento inverso)?</span><span class="sxs-lookup"><span data-stu-id="d19c0-116">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="d19c0-117">Em caso afirmativo, considere o uso do Azure HDInsight com Spark MLlib e Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="d19c0-117">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="d19c0-118">Você precisa de funcionalidades de NLP simples e de alto nível, como identificação de entidade e intenção, detecção de tópico, verificação ortográfica ou análise de sentimento?</span><span class="sxs-lookup"><span data-stu-id="d19c0-118">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="d19c0-119">Em caso afirmativo, considere o uso das APIs oferecidas pelos Serviços Cognitivos da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="d19c0-119">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="d19c0-120">Matriz de funcionalidades</span><span class="sxs-lookup"><span data-stu-id="d19c0-120">Capability matrix</span></span>

<span data-ttu-id="d19c0-121">As tabelas a seguir resumem as principais diferenças em funcionalidades.</span><span class="sxs-lookup"><span data-stu-id="d19c0-121">The following tables summarize the key differences in capabilities.</span></span>

### <a name="general-capabilities"></a><span data-ttu-id="d19c0-122">Funcionalidades gerais</span><span class="sxs-lookup"><span data-stu-id="d19c0-122">General capabilities</span></span>

| | <span data-ttu-id="d19c0-123">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="d19c0-123">Azure HDInsight</span></span> | <span data-ttu-id="d19c0-124">Serviços Cognitivos da Microsoft</span><span class="sxs-lookup"><span data-stu-id="d19c0-124">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="d19c0-125">Fornece modelos pré-treinados como serviço</span><span class="sxs-lookup"><span data-stu-id="d19c0-125">Provides pretrained models as a service</span></span> | <span data-ttu-id="d19c0-126">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-126">No</span></span> | <span data-ttu-id="d19c0-127">SIM</span><span class="sxs-lookup"><span data-stu-id="d19c0-127">Yes</span></span> |
| <span data-ttu-id="d19c0-128">API REST</span><span class="sxs-lookup"><span data-stu-id="d19c0-128">REST API</span></span> | <span data-ttu-id="d19c0-129">SIM</span><span class="sxs-lookup"><span data-stu-id="d19c0-129">Yes</span></span> | <span data-ttu-id="d19c0-130">SIM</span><span class="sxs-lookup"><span data-stu-id="d19c0-130">Yes</span></span> |
| <span data-ttu-id="d19c0-131">Programação</span><span class="sxs-lookup"><span data-stu-id="d19c0-131">Programmability</span></span> | <span data-ttu-id="d19c0-132">Python, Scala, Java</span><span class="sxs-lookup"><span data-stu-id="d19c0-132">Python, Scala, Java</span></span> | <span data-ttu-id="d19c0-133">C#, Java, Node.js, Python, PHP, Ruby</span><span class="sxs-lookup"><span data-stu-id="d19c0-133">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="d19c0-134">Dá suporte ao processamento de conjuntos de Big Data e documentos grandes</span><span class="sxs-lookup"><span data-stu-id="d19c0-134">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="d19c0-135">SIM</span><span class="sxs-lookup"><span data-stu-id="d19c0-135">Yes</span></span> | <span data-ttu-id="d19c0-136">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-136">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="d19c0-137">Funcionalidades de processamento de idioma natural de baixo nível</span><span class="sxs-lookup"><span data-stu-id="d19c0-137">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="d19c0-138">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="d19c0-138">Azure HDInsight</span></span> | <span data-ttu-id="d19c0-139">Serviços Cognitivos da Microsoft</span><span class="sxs-lookup"><span data-stu-id="d19c0-139">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- |
| <span data-ttu-id="d19c0-140">Gerador de token</span><span class="sxs-lookup"><span data-stu-id="d19c0-140">Tokenizer</span></span> | <span data-ttu-id="d19c0-141">Sim (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d19c0-141">Yes (Spark NLP)</span></span> | <span data-ttu-id="d19c0-142">Sim (API de Análise Linguística)</span><span class="sxs-lookup"><span data-stu-id="d19c0-142">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="d19c0-143">Lematizador</span><span class="sxs-lookup"><span data-stu-id="d19c0-143">Stemmer</span></span> | <span data-ttu-id="d19c0-144">Sim (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d19c0-144">Yes (Spark NLP)</span></span> | <span data-ttu-id="d19c0-145">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-145">No</span></span> |
| <span data-ttu-id="d19c0-146">Lematizador</span><span class="sxs-lookup"><span data-stu-id="d19c0-146">Lemmatizer</span></span> | <span data-ttu-id="d19c0-147">Sim (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d19c0-147">Yes (Spark NLP)</span></span> | <span data-ttu-id="d19c0-148">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-148">No</span></span> |
| <span data-ttu-id="d19c0-149">Marcação de parte do discurso</span><span class="sxs-lookup"><span data-stu-id="d19c0-149">Part of speech tagging</span></span> | <span data-ttu-id="d19c0-150">Sim (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d19c0-150">Yes (Spark NLP)</span></span> | <span data-ttu-id="d19c0-151">Sim (API de Análise Linguística)</span><span class="sxs-lookup"><span data-stu-id="d19c0-151">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="d19c0-152">TF/IDF (frequência de termo/frequência de documento inverso)</span><span class="sxs-lookup"><span data-stu-id="d19c0-152">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="d19c0-153">Sim (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d19c0-153">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d19c0-154">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-154">No</span></span> |
| <span data-ttu-id="d19c0-155">Similaridade de cadeia de caracteres &mdash; editar cálculo de distância</span><span class="sxs-lookup"><span data-stu-id="d19c0-155">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="d19c0-156">Sim (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d19c0-156">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d19c0-157">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-157">No</span></span> |
| <span data-ttu-id="d19c0-158">Cálculo de N-gram</span><span class="sxs-lookup"><span data-stu-id="d19c0-158">N-gram calculation</span></span> | <span data-ttu-id="d19c0-159">Sim (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d19c0-159">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d19c0-160">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-160">No</span></span> |
| <span data-ttu-id="d19c0-161">Remoção de palavra irrelevante (stop word)</span><span class="sxs-lookup"><span data-stu-id="d19c0-161">Stop word removal</span></span> | <span data-ttu-id="d19c0-162">Sim (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d19c0-162">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d19c0-163">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-163">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="d19c0-164">Funcionalidades de processamento de idioma natural de alto nível</span><span class="sxs-lookup"><span data-stu-id="d19c0-164">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="d19c0-165">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="d19c0-165">Azure HDInsight</span></span> | <span data-ttu-id="d19c0-166">Serviços Cognitivos da Microsoft</span><span class="sxs-lookup"><span data-stu-id="d19c0-166">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="d19c0-167">Identificação de entidade/intenção e extração</span><span class="sxs-lookup"><span data-stu-id="d19c0-167">Entity/intent identification & extraction</span></span> | <span data-ttu-id="d19c0-168">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-168">No</span></span> | <span data-ttu-id="d19c0-169">Sim [API de LUIS (Serviço Inteligente de Reconhecimento Vocal)]</span><span class="sxs-lookup"><span data-stu-id="d19c0-169">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |
| <span data-ttu-id="d19c0-170">Detecção de tópico</span><span class="sxs-lookup"><span data-stu-id="d19c0-170">Topic detection</span></span> | <span data-ttu-id="d19c0-171">Sim (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d19c0-171">Yes (Spark NLP)</span></span> | <span data-ttu-id="d19c0-172">Sim (API de Análise de Texto)</span><span class="sxs-lookup"><span data-stu-id="d19c0-172">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="d19c0-173">Verificação ortográfica</span><span class="sxs-lookup"><span data-stu-id="d19c0-173">Spell checking</span></span> | <span data-ttu-id="d19c0-174">Sim (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d19c0-174">Yes (Spark NLP)</span></span> | <span data-ttu-id="d19c0-175">Sim (API de Verificação Ortográfica do Bing)</span><span class="sxs-lookup"><span data-stu-id="d19c0-175">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="d19c0-176">Análise de sentimento</span><span class="sxs-lookup"><span data-stu-id="d19c0-176">Sentiment analysis</span></span> | <span data-ttu-id="d19c0-177">Sim (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d19c0-177">Yes (Spark NLP)</span></span> | <span data-ttu-id="d19c0-178">Sim (API de Análise de Texto)</span><span class="sxs-lookup"><span data-stu-id="d19c0-178">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="d19c0-179">Detecção de idioma</span><span class="sxs-lookup"><span data-stu-id="d19c0-179">Language detection</span></span> | <span data-ttu-id="d19c0-180">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-180">No</span></span> | <span data-ttu-id="d19c0-181">Sim (API de Análise de Texto)</span><span class="sxs-lookup"><span data-stu-id="d19c0-181">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="d19c0-182">Dá suporte a vários idiomas além do inglês</span><span class="sxs-lookup"><span data-stu-id="d19c0-182">Supports multiple languages besides English</span></span> | <span data-ttu-id="d19c0-183">Não </span><span class="sxs-lookup"><span data-stu-id="d19c0-183">No</span></span> | <span data-ttu-id="d19c0-184">Sim (varia de acordo com a API)</span><span class="sxs-lookup"><span data-stu-id="d19c0-184">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="d19c0-185">Consulte também</span><span class="sxs-lookup"><span data-stu-id="d19c0-185">See also</span></span>

[<span data-ttu-id="d19c0-186">Processamento de idioma natural</span><span class="sxs-lookup"><span data-stu-id="d19c0-186">Natural language processing</span></span>](../scenarios/natural-language-processing.md)
