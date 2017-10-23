---
layout: post
title: Notes of QCon Shanghai 2017
tags: [Conference]
---

## General thoughts


1. If a simple algorithm can solve the problem, do we need a more complex algorithm to implement it?
2. Most companies use multiple models to solve a single problem.
3. Similar to software development, algorithm development needs to iterate as well: model update and feature engineering.
3. When the scale is growing, we may need build a platform to support more algorithm developments.
4. The evaluation of online inference may be different from the one of offline testing. We need align two evaluations.
5. Split online training and offline training
6. Do not use micro-service for micro-service. Implement with different gratitude.

## Sessions

### Session 1 by Prof. Hui WEI from Fudan

**Theme: AI**

**Takeaway: We need treat AI carefully.**

Limited definitions of of AI:

  1. based on code
  2. can be defined literally
  3. can be defined by algorithms
  4. can be computed

Why can Go game be solved by algorithms?

  1. clean data
  2. limited rules
  3. huge amount of tagged data

**Unsolved question: how does knowledge guide our action without programmed routines?**

Possible direction: a light-weight solution to do inference

Readings:
> David Kirsh, Foundations of AI: The Big Issues. *Artificial Intelligence*. (1991). [http://adrenaline.ucsd.edu/kirsh/Articles/BigIssues/big-issues.pdf][1]

### Session 2 by Zaiqing NIE from Alibaba

**Theme: AliGene**

**Takeaway: An easy-to-use NLP platform is a new trend.**

An HCI platform based on NLP

Request \> Intent detection \> Slot filling (knowledge base, user profiling) \> dialogue management (predefined) \> Response

Challenges:

  1. Ambiguity of natural language
  2. limited tagged data
  3. high expectation of precision
  4. limited NLP developers

Solutions:

  Cold start \< rules
  Warm start \< deep learning
  Reinforcement learning
  Active learning (Use questions to increase certainty)

Readings:

> Agichtein, E., & Gravano, L. (2000, June). Snowball: Extracting relations from large plain-text collections. In *Proceedings of the fifth ACM conference on Digital libraries* (pp. 85-94). ACM. [http://www.mathcs.emory.edu/%7Eeugene/papers/dl00.pdf][2]

### Session 3 by Hui Wang from Paypal

**Theme: Risk Management by Using AI**

**Takeaway: A single problem might need multiple models and rules**

Risk management has no dimension limit, no volume limit for AI.

Data sources:

  1. account behaviors
  2. sessions (visiting histoy)
  3. negative trading history
  4. operational data
  5. external data

Detection order:

  1. transactions
  2. account level data
  3. multi account level (related accounts)

Story-based approach to decrease false positives
> Run multiple models and rules before making decisions on transactions - all within milliseconds.


### Session 4 by Xiongjie LIAO from TalkingData

**Theme: Monitoring Micro-service**

**Takeaway: Monitoring only the key metrics is be more effective and efficient.**

Micro-service split-up:

  1. horizontal
  2. vertical
  3. hybrid

Problems of micro-service:

  1. deployment management
  2. complicated workflow
  3. complicated monitoring

Approached of monitoring:

  1. focusing on performance (lightweight)
  2. focusing on workflow (currently main stream)

Philosophy of TalkingData’s solution:

  1. performance monitoring first
  2. only monitor possible bottlenecks (events VS workflows)
  3. locate the specific code caused bottlenecks

Key metrics to monitor:

  1. API response time
  2. throughput
  3. API response components
  4. network IO
  5. API workflow time (can use [dubbo][3] to monitor)
  6. slow request stacks


### Session 6 by Yuefeng ZHOU from Google Brain

**Theme: Tensorflow as a Computing Framework**

**Takeaway: We can use machine learning to optimize machine model / algorithms/ computing.**

New features:

  1. Eager Execution
  2. 2nd TPU
  3. Tensorflow APIs (Keras 2.0)
  4. Estimator
  5. Tensorflow Lite
	1. on mobile device
	2. offline inference
	3. APM-optimised
  6. New Input Pipeline
	1. input pipeline = lazy lists
	2. map + filter
	3. tf.data
	  1. Dataset \> functional programming
	  2. Iterator \> sequential input

Learn2learn algorithms for selecting the best models

  1. based on RNN
  2. evolution algorithms
	1. select the best as parent
	2. pick two randomly
	3. drop the lower one
	4. copy mutate  parents

Reinforcement learning for device placement

  1. many-device training (distributed)
  2. design for bigger models
  3. design for larger batch size


### Session 7 by Yunsong GUO from Pinterest

**Theme: Homefeed Recommendation in Pinterest**

**Takeaway:**

  1. Feature engineering may bring higher returns than applying new models.
  2. Iterate models / features is a possible way to increase performance.
  3. A simple algorithm may perform better than a complicated model.
  4. Iterate quickly.
  5. Online inference may have a different evaluation with the offline training.

Features to use:

  1. users
  2. pins
  3. interactions

Workflow:

  1. candidate generation (reduce data amount)
	1. collaborative filtering (use different pictures from same boards)
	2. random walking
  2. machine learning algorithms
	1. time based
	2. logistic regressions
	  1. LR
	  2. SVM (sofia-ml)
	  3. feature engineering
		1. the best feature may increase 4% relevance,
		2. age is not a useful feature in this case
	  4. GBDT
		1. use with ensambling model
		2. split sparse and non-sparse features first
		3. similar effect as word-embedding
  3. feed generation (may reduce relevance)

Feature engineering

  1. specialized \< needs to train internal domain experts
  2. high failure rate \< be patient, iterate faster, reduce deployment period
  3. low transparency \< record experiments
  4. high labor cost \< build a platform to support
  5. high return \< higher than new models

Cold start problem:

  1. ask questions
  2. use data from similar users

Evaluation difference of online inference and offline training:

  1. find the correlation between online and offline standards
  2. map the offline standard to online one.
  3. reduce experiment period
  4. filter out low-return experiments

Diversity problem:

  1. use recommendations from different categories
  2. drop some relevance for diversity

### Session 8 by Lou YIN from Airbnb

**Theme: GB Decision Table**

**Takeaway: Data structure may have a huge impact on performance.**

Best practices:

  1. set depth to a limited number
  2. set shinkage
	1. default is 1
	2. better use a smaller one, e.g.: 0.01
  3. sub-sampling
	1. similar to SGD
  4. tradeoff of model size
	1. accuracy ↔ scoring time

Decision table VS decision tree

  1. freedom: decision table \< decision tree
  2. speed: decision table \> decision tree (50% gain)
  3. variance and bias: decision table \< decision tree

Data  structure of decision table

Optimization of decision table in gradient boosting

1. backfitting:
	1. cyclic (recommended)
	2. random (recommended)
	3. greedy (biggest gain, but slow)
 2. speed up scoring
	1. avoid repeated tests
	2. sort the table by values
	  3. online inference
		1. pre-set user features

Feature engineering of GBDT

1. feature discretisation
	1. reduce outliers
	2. euqi-width
	3. equi-frequency
	4. not good for linear features (pre-compute a linear model)

Readings:
> Lou, Y., & Obukhov, M. (2017). BDT: Gradient Boosted Decision Tables for High Accuracy and Scoring Efficiency (pp. 1893–1901). Presented at the the 23rd ACM SIGKDD International Conference, New York, New York, USA: ACM Press. https://yinlou.github.io/papers/lou-kdd17.pdf



### Session9 by Eric KIM from LinkedIn

**Theme: Helix and Nuage**

**Takeaway:**

  1. Easy-to-use and operatibility should be planned in advance.
  2. Applying algorithms in engineering is a new trend.

Helix: data storing platform
Nuage: data consuming platform

Invisible:

  1. ultra-simple to use
  2. elastic and scale easily
  3. no single point of failure
  4. highly operatable

challenges:

1. fault-tolerance
2. data replication
3. job management
4. load balancing
5. high performance
6. failure fix

capacity detection:

  1. pessimistic model
  2. ARIMA


### Session 12 by Zhiyuan ZONG from QIY

**Theme: Risk Management in QIY**

**Takeaway: Always clarify and focus on the needs.**

High precision, low recall
Split account behavior from textual data
Use LSTM to predict user behaviors (sequence prediction)


### Session 13 by Ming HUANG from Tencent

**Theme: Spark on Angel**

**Takeaway: Modify a tool if needed.**

Needs: solve big models on big data

  1. driver is the main bottleneck
  2. needs to reduce dimension
  3. executors  need to wait (PST mode)

Architecture

1. mutable layer on immutable layer
2. PS model
	1. mutable
	2. no new memory allocated
	3. operate PS model in order to operate on remote servers
3. server: flexible
4. model layer: virtual
5. client: replaceable with DL / spark

Angel API

1. simplify writing procedure
2. a unifier API
	1. vector (inherit from Spark: Breeze PS, Cached PS)
	2. MLLib in Spark is based on Breeze
	3. easy to migrate to Angel
	4. matrix

### Session 14 by Chao Can from Amazon

** Theme: Micro-service and Serverless**

** Takeaway**

1. Refactoring to micro-service is an option.
2. Consider about data structure when doing refactoring.
3. Micro-service has different gratitude, choose accordingly.

The proper team size in Amazon is about the number that can share two pizzas.

Refactoring:

1. make code as modules
2. interface
3. make facade
4. split database
5. build local mock

Auto-scaling is not at run time, because it needs time to load.

Moving to server less

1. benchmark everything
2. functions should be stateless
3. user [AWS Step Functions][4]
4. be careful with FAAS (function as a service)
5. keep it natural
	use [CLI][5] to control [Lambda][6]

Readings:
[Domain-Driven Design][7]
[The Art of Scalibility][8]

### Session 15 by Youlin LI from Facebook

** Theme: Real-time training of Newsfeed**

**Takeaway: Follow the defined goal to develop and iterate.**

Slogan of Facebook:

1. Focus on impact
2. Move fast

Daily deployment

FB Joiner captures all events for a given story/session within a window, and outputs.

Newsfeed:

1. rule-based ranking -\> machine-learning-based ranking
2. use feedback to retrain model
3. real-time joining
4. time window: 3 min (invalid if a session  is longer than 3 mins)
5. tolerate value losses (0.5% loss)





[1]:	http://adrenaline.ucsd.edu/kirsh/Articles/BigIssues/big-issues.pdf
[2]:	http://www.mathcs.emory.edu/%7Eeugene/papers/dl00.pdf
[3]:	(https://github.com/alibaba/dubbo
[4]:	https://aws.amazon.com/step-functions/
[5]:	https://aws.amazon.com/cli/
[6]:	https://aws.amazon.com/lambda/
[7]:	https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215
[8]:	https://www.amazon.com/Art-Scalability-Architecture-Organizations-Enterprise/dp/0134032802/
