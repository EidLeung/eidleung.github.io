---
layout:      post
classify:    "GitChat"
title:       "领域驱动设计-战略篇之领域知识"
subtitle:    "系统全面详尽讲解领域驱动设计"
date:        2019-11-22
catalog:     true
author:      "EidLeung"
tags:
    - GitChat
    - DDD
    - 战略
---

<center><h1><b>领域知识</b></h1></center>
<p align="right"><a href="#如何购买">购买专栏</a></p>

## 01. 团队沟通与写作
### 1. 需求偏差
1. 我们从客户那里了解到的需求，并非用户最终的需求；
2. 若无有效的沟通方式，需求的理解偏差则会导致结果大相径庭；
3. 理解到的需求并没有揭示完整的领域知识，从而导致领域建模与设计出现认知障碍。

### 2. 团队协作
#### 2.1. 先启阶段
![先启阶段](/img/gitchat/DDD/xqjd.png)
1. 首先，我们需要确定项目的利益相关人，并通过和这些利益相关人的沟通，来确定系统的业务期望与愿景。
2. 然后，在期望与愿景的核心目标指导下，团队与客户才可能就问题域达成共同理解。
3. 这时，我们需要确定项目的当前状态与未来状态，从而确定项目的业务范围。之后，就可以对需求进行分解了。
4. 在先启阶段，对需求的分析不宜过细，因此需求分解可以从史诗级（Epic）到主故事级（Master）进行逐层划分，并最终在业务范围内确定迭代开发需要的主故事列表。

#### 2.2. 迭代开发阶段
![迭代开发阶段](/img/gitchat/DDD/ddkfjd.png)

##### 2.2.1. Scrum 敏捷管理的过程（会议）
###### 2.2.1.1. 计划会议
1. 产品负责人（Product Owner）在会议中向团队成员介绍和解释该迭代需要完成的用户故事，包括用户故事的业务逻辑与验收标准。
2. 团队成员对用户故事有任何不解或困惑，都可以通过这个会议进行沟通，初步达成领域知识的共识。

###### 2.2.1.2. 站立会议
1. 产品负责人参与并及时解答开发过程中可能出现的需求理解问题
2. Scrum Master 则通过每天的站立会议了解当前的迭代进度，并与产品负责人一起基于当前进度和迭代目标确定是否需要调整需求的优先级

###### 2.2.1.3. 迭代演示会议
除开发团队，还可以邀请客户、最终用户以及领域专家参与，由团队的测试人员演示当前迭代已经完成的功能。消除用户、客户、领域专家、产品负责人与团队在需求沟通与理解上的偏差。

##### 2.2.2. Scrum 敏捷管理的关键节点
###### 2.2.2.1. Kick Off——从需求到开发
在开发人员领取了用户故事，并充分理解了用户故事描述的需求后，将需求分析人员与测试人员叫过来，大家一起做一个极短时间的沟通与确认。

###### 2.2.2.2. Desk Check——从开发到测试
当开发完成后，开发环境下，由开发人员向需求分析人员与测试人员“实地”演示刚刚完成的功能，并对照着验收标准进行验收。

## 02.（案列）先启阶段
> 先启阶段的需求分析重点是对史诗级故事和主故事的识别与拆分。

1. 首先从调用者（参与角色）的角度出发（Actor），去分析需求功能、识别需求的价值，并找到系统的边界。
2. 与外部团队的协作存在较大风险。一些功能的开发需要与其他团队协作，若能将这部分功能的迭代周期提前，将有利于我们更好地与其他团队进行协作。
3. 在需求分析时，我们将自己的视角切换到参与者的观察角度，可以得出属于该参与者的主要用例，这些用例就成为了主故事列表的重要输入。

## 03. 从领域场景到领域知识（上）
### 1. 领域场景分析的方法
`用例的关注点就是领域。用例表达的领域概念必须精准！`
用例尤其是用例图的抽象能力更强，更擅长于对系统整体需求进行场景分析

1. Who——参与者（Actor）
2. What——用例（Use Case），明确业务功能
3. Why——用例关系：包括使用、包含、扩展、泛化、特化等关系，其中使用（use）关系，这一功能给该角色能够带来什么样的业务价值
4. Where——边界（Boundary）
5. When——表明时机，何时被调用
6. hoW——子用例，表明实现方式

## 04. 从领域场景到领域知识（下）
1. **角色**支持对产品功能的细分，而且它经常引出其他角色的需要以及相关
2. **活动**的环境；活动通常表述相关角色所需的“系统需求”；
3. **价值**则传达为什么要进行相关活动，也经常可以引领团队寻找能够提供相同价值而且更少工作量的替代活动。

### 1. 用户故事
1. 用户故事提供了场景分析的固定模式，善于表达具体场景的业务细节。
2. 一个完整的用户故事必须是可测试（Testable）的，因此验收标准（Acceptance Criteria）是用户故事不可缺少的部分。
3. 我们在编写用户故事时，应该按照行为驱动开发的要求，关注于做什么（what），而不是怎么做（how）。
4. 用户故事应该只受到业务规则与业务流程变化的影响。

### 2. 测试驱动开发
1. 测试驱动开发则强调对业务的分解，利用编写测试用例的形式驱动领域建模，即使不采用测试先行，让开发者转换为调用者角度去思考领域对象及行为，也是一种很好的建模思想与方法。
2. 试驱动开发强调“测试优先”，其实质是需求分析优先，是任务分解优先。
3. 不应针对被测方法编写单元测试，而应该根据领域场景进行编写，站在调用者的角度去思考。

#### 3.1 . Given-When-Then 模式
1. 编写Given  
“驱动”我们思考被测对象的创建，以及它与其他对象的协作。
2. 编写When  
“驱动”我们思考被测接口的方法命名，以及它需要接收的传入参数；考虑行为方式，究竟是命令式还是查询式方法。  
可以帮助开发者思考类的行为，一定要从业务而非实现的角度去思考接口。
3. 编写Then  
“驱动”我们分析被测接口的返回值。  
考虑的是如何验证，没有任何验证的测试不能称其为测试。

## 05. 建立统一的语言
> 需求分析的过程，团队中各个角色就系统目标、范围与具体功能达成一致。

### 1. 建立统一的领域术语
1. 形成统一的领域术语，尤其是基于模型的语言概念，是沟通能够达成一致的前提。
2. 在维护领域术语表时，一定需要给出对应的英文术语，否则可能直接影响到代码实现。

### 2. 用统一的术语进行领域行为描述
1. 从领域的角度而非实现角度描述领域行为
2. 若涉及到领域术语，必须遵循术语表的规范
3. 强调动词的精确性，符合业务动作在该领域的合理性
4. 要突出与领域行为有关的领域概念

---
###### 如何购买
1. 扫描二维码
	<div align="center">
		<a href="http://gitbook.cn/m/mazi/comp/column?columnId=5cdab7fb34b6ed1398fd8de7&sceneId=f52a3810af7511e992061554a506c6dc&utm_source=columninvitecard&utm_campaign=%E9%A2%86%E5%9F%9F%E9%A9%B1%E5%8A%A8%E8%AE%BE%E8%AE%A1%E5%AE%9E%E8%B7%B5%E5%90%88%E8%AE%A2%E7%89%88%EF%BC%88%E6%88%98%E7%95%A5%2B%E6%88%98%E6%9C%AF%EF%BC%89">
			<img src="/img/gitchat/DDD/DDD.jpg" width = "300" height = "300" alt="图片名称" style="display: inline-block"/>
		</a>
		<img src="/img/JTYK.jpg" width = "300" height = "300" alt="今天有课" style="display: inline-block"/>
	</div>
2. 点击链接  
[极客时间-直连](http://gitbook.cn/m/mazi/comp/column?columnId=5cdab7fb34b6ed1398fd8de7&sceneId=f52a3810af7511e992061554a506c6dc&utm_source=columninvitecard&utm_campaign=%E9%A2%86%E5%9F%9F%E9%A9%B1%E5%8A%A8%E8%AE%BE%E8%AE%A1%E5%AE%9E%E8%B7%B5%E5%90%88%E8%AE%A2%E7%89%88%EF%BC%88%E6%88%98%E7%95%A5%2B%E6%88%98%E6%9C%AF%EF%BC%89)  
[今天有课-红包](https://jika.nali.net/youke/coupon/getCouponList?sendUserId=17140)