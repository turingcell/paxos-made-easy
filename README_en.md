## Paxos Made Easy: The Geometric Meaning and Geometric Proof of Paxos Algorithm

![TuringCell Logo](img/logo_small_icon.png)

This is a sub-project of open-source project [TuringCell](https://turingcell.org). Turing Cell Model is a computing model running on top of distributed consistency algorithms (such as Paxos/Raft). TuringCell is an open-source implementation of Turing Cell Model. This means that you can add features as high-availability, fault-tolerance and strong-consistency to existing software very easily. At the same time, TuringCell is an industry-friendly project, at its core is the force from an open, tolerant community. Wherever you are from, whichever language you speak, you are all welcome to equally join in, discuss and build TuringCell !

Author: Sen Han 00hnes@gmail.com

Translator: Yicheng Liu  liuyicheng01@outlook.com

# Table of Content

   * [0 Abstract](#0-Abstract)
   * [1 Preface](#1-Preface)
   * [2 Getting to Know Paxos](#2-Getting-to-Know-Paxos)
     * [2.1 System Assumptions](#21-System-Assumptions)
     * [2.2 Scene Definition](#22-Scene-Definition)
     * [2.3 Acceptor Definition](#23-Acceptor-Definition)
     * [2.4 Proposer Definition](#24-Proposer-Definition)
   * [3 Geometric Meaning and Proof](#3-Geometric-Meaning-and-Proof)
     * [3.1 Definitions](#31-Definitions)
     * [3.2 Theorems](#32-Theorems)
     * [3.3 Mathematical Proof](#33-Mathematical-Proof)
     * [3.4 Geometric Proof](#34-Geometric-Proof)
     * [3.5 Others](#35-Others)
   * [4 Summary](#4-Summary)
   * [5 Philosophical Meaning Behind Mathematical Proof](#5-Philosophical-Meaning-Behind-Mathematical-Proof)
   * [6 Community, Support and Cooperation](#6-ommunity,-Support-and-Cooperation)
   * [7 Donation](#7-Donation)
   * [8 Update History](#8-Update-History)
   * [9 Copyright and License](#9-Copyright-and-License)

### 0 Abstract

This paper describes the geometric meaning of Paxos algorithm and proposes a novel yet strict proof method called 'distributed time-line diagram', which is a very simple and vivid mathematical proof method. The actual proof part in this paper is short, most of which is attempting to strictly and clearly define the Paxos algorithm. Therefore, this paper can also be a study document for newers to learn Paxos algorithm.

### 1 Preface

There are plenty of classical algorithms in computer science field. However, some algorithms including those having won the Turing Prize might bring us the feeling 'I would have come up with the similar great algorithm, if I was facing the same issue at that time when computer science was advancing rapidly'. Paxos algorithm would never be in this list.

Unimaginably, such a simple, graceful and precise algorithm has solved one of the most difficult problems in distributed system. At the same time, its mathematical proof is also ingenious and intoxicating. It feels like taking a quick glance at a gorgeous lady without intention yet would never be able to forget for the rest of life.

Paxos at its core is very simple and direct as well as the optimization in real application -- as long as satisfying foundational constraints it needs, we are free to customize to optimize its performance. 

However, to implement an algorithm at productive environment flawlessly, one has to have fully understood it and proven it using sense and logics independently and strictly. One also had better perceive the algorithm directly through the senses, otherwise would be confused when dealing with different abnormal situations and optimization for special scenes. You could only pray to God for bugfree if you failed to fully understand the algorithm but still longed for correctness. This is also the reason why this paper was delivered.

Even though Paxos is so simple and easy to prove, why are there a lot of people still regarding it hard to understand? There are two main reasons in my opinion:

1. Distributed systems have their internal complexity, which is not comparable to the complexity of single node running model system. This can be regarded as an '[emergence](https://zh.wikipedia.org/zh/%E6%B6%8C%E7%8E%B0)'.
2. A lot of documents tend to teach Paxos algorithm theory and proof using a perceptual, blurry even comparative way. We think that this sometimes might make things even worse. Just like the best way to learn math is to deduce each theorem independently instead of repeating them, which only makes the study harder to understand. Paxos is precise but not simple, the first step to understand it is to define it using strict mathematical description, that is, to define clearly a mathematical problem. Only when a problem has been defined clearly without ambiguity can we declare that we have sorted out what exactly this problem is, then the proof may come out.

If you are very familiar with Paxos algorithm and able to prove it, you can just jump to [Section 3](#3-几何意义与证明).

If you are not able to prove the algorithm independently, even though familiar with the algorithm, you should also read the paper carefully.

That is because one can hardly understand how the algorithm runs and its hidden essential constraint condition fully if not able to prove its correctness precisely, also not be easy to customize and optimize the algorithm in real production environment. Therefore, this paper is not only a proof, most of the contents at the beginning are making a precise engineering description to the algorithm. If you find it different from what you learned before, that's good, for which that is where the problem lies. After reading the paper for the first time, if your goal has not been achieved, read again, please. If your goal has not been achieved still after second reading, please send your questions and doubts in emails to me, I will try to reply as soon as possible.

**All the discussions are under the assumption that '[Byzantine Fault](https://zh.wikipedia.org/wiki/%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98) doesn't exist in the system'**

This paper is under the version control in [this repo](https://github.com/turingcell/paxos-made-easy), new issues and PRs are very welcome :)

### 2 Getting to know Paxos

#### 2.1 System Assumptions 

Assumptions that exist in common distributed system running models:

> &emsp;A0.1&emsp;Processors operate at arbitrary speed
>
> &emsp;A0.2&emsp;Processors may experience failures
>
> &emsp;A0.3&emsp;Processors with stable storage may re-join the protocol after failures
>
> &emsp;A0.4&emsp;Processors do not collude, lie, or otherwise attempt to subvert the protocol. That is, Byzantine failures don't occur
>
> &emsp;A0.5&emsp;Processors can send messages to any other processor
>
> &emsp;A0.6&emsp;Messages are sent asynchronously and may take arbitrarily long to deliver
>
> &emsp;A0.7&emsp;Messages may be lost, reordered, or duplicated
>
> &emsp;A0.8&emsp;Messages are delivered without corruption. That is, Byzantine failures don't occur
>
> &emsp; -- https://en.wikipedia.org/wiki/Paxos_(computer_science)#Assumptions

We will get the default system assumptions in this paper if we replace the assumption A0.7 as A0.9:

&emsp;**A0.9**&emsp;**Messages may be lost, reordered, but will never be duplicated**

That means the assumptions do not include the situation where 'message duplication exists' in this paper. Even though it is obvious and not necessary, this paper extends the proof from the assumption that 'message without duplication' to the common distributed system assumption that 'message with duplication'. More details please refer [Section 3.5](#35-琐碎事项)

#### 2.2 Scene Definition

![paxos_timeline_new_proof_1](img/paxos_timeline_new_proof_1.jpg)

Figure 1 describes a Paxos instance where there are five Acceptors and arbitrary numbers of Proposers. At the moment when *t=0*, all Acceptors are at initialized states.

Acceptor is a critical role in Paxos running instance. Once paxos running instance includes a fixed set of Acceptors, which we call a *configuration* of a Paxus running instance:

![latex1](img/latex1.png)

For the Paxos instance where there is only one Acceptor, its configuration can be described as follows:

![latex2](img/latex2.png)

The configuration of Paxos instance in Figure 1 can be described as:

![latex3](img/latex3.png)

The system satisfies the following constraints at any circumstance:

&emsp;&emsp;**Constraint 0.1**&emsp;**One Paxos instance has and only has one fixed configuration**

In the figure above, when the system is at time *t<0*, one Paxos instance was created with configuration *config0*, which would never be mutated; At moment *t=0*, all Acceptors have finished initialization and been ready to provide service.

In addition, not only the role Acceptor is defined in one Paxos instance (the one fixed configuration), but also another role called Proposer. There is no limit on the number of Proposers, they can be regarded as the client to access Acceptors. When *t>0*, Proposers send requests called *Prepare* and *Accept* to Acceptors inside the configuration.

#### 2.3 Acceptor Definition

Each Acceptor in normal state maintains a persistently stored state stable *t*. Take Acceptor0 as an example, state table *t* is as follows:

```lua
-- initial value of t
local t = { 
    id  = "acceptor0", -- constant
    cfg = config0,     -- config of paxos
    p_e = 0,           -- prepare_epoch
    a_e = 0,           -- accept_epoch
    a_v = nil          -- accept_value
}
```

* *id* represents the identification of this Acceptor, which is read-only;
* *cfg* is the configuration value of this Paxos instance, which is read-only;
* *p_e* is a non-negative integer with arbitrary degree of accuracy, it has the initial value of 0;
* *a_e* is a non-negative integer with arbitrary degree of accuracy, it has the initial value of 0;
* *a_v* is a binary string with arbitrary length, it has the initial value of *nil*.

Acceptor provides Proposer with two RPC services: Prepare (abbr. P) and Accept (abbr. A).

```lua
P(e) -> { true, t.p_e, t.a_e, t.a_v } || { false, t.p_e }
```

```lua
-- serve input RPC: P(e)
local function acceptor_prepare_srv(e)
    byzantine_assert( t.p_e >= 0 and t.p_e >= t.a_e )
    if e > t.p_e then
        t.p_e = e
        must_success_fsync_t()
        return { true, t.a_e, t.a_v }
    else
        return { false, t.p_e }
    end
end
```

When an Acceptor receives a *P(e)* request, it calls function _acceptor\_prepare\_srv(e)_ to handle and reply.

*byzantine\_assert* is called Byzantine Fault Assertion, which runs a couple of error checks on basic logics, if errors are found, it notifies all the Acceptors in the Paxos configuration set to shut down. Due to one of the prerequisites of Paxos algorithm that no Byzantine fault exists in the system, obviously it makes no sense if the system is forced to continue, which may cause worse issues. All these logical checks are evaluated as true in normal cases unless there are severe bugs or wrong calculations, for more precise definitions and discussions please refer to [Byzantine Generals Problem](https://zh.wikipedia.org/wiki/%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98) and [Byzantine Paxos Algorithm](https://lamport.azurewebsites.net/tla/byzsimple.pdf). Byzantine fault checks do not belong to Paxos algorithm, you can ignore it if you would like to.

Function *must\_success\_fsync\_t* stores the state of Acceptor to persistent disks, only if this function returns with no error can the request be continued to process; If the function fails, Acceptor will exit with exception.

**Semantics of _P(e)_ **:

&emsp;**After Acceptor receives *P(e)* request，it checks if *e > t.p\_e* is true**

&emsp;&emsp;**• true: assignment *t.p\_e = e* is run, state table *t* is updated, then *t* is persistently stored, at last function returns _\{ true, t.a\_e, t.a\_v \}_**

&emsp;&emsp;**• false, function returns _\{ false, t.p\_e \}_**

&emsp;**If an Acceptor receives multiple P(e) requests simultaneously, the Acceptor guarantees that each P(e) request is processed strictly serially, independently and atomically**.

```lua
A(e,v) -> { true } || { false, t.p_e }
```

```lua
-- serve input RPC: A(e,v)
local function acceptor_accept_srv(e,v)
    byzantine_assert( t.p_e >= 0 and t.p_e >= t.a_e and e <= t.p_e )
    if e == t.p_e then
        if e == t.a_e then
            byzantine_assert(t.a_v == v)
            return { true }
        else
            t.a_e = e
            t.a_v = v
            must_success_fsync_t()
            return { true }
        end
    else
        return { false, t.p_e }
    end
end
```

When an Acceptor receives *A(e,v)* request, it calls *acceptor\_accept\_srv* function to handle and reply.

**Semantics of _A(e,v)_ :**

&emsp;**After Acceptor receives *A(e,v)* request, it checks condition *e == t.p\_e***

&emsp;&emsp;**• true: checks condition *e == t.a\_e***

&emsp;&emsp;&emsp;**・ true: runs *byzantine\_assert(t.a\_v == v)* , returns _\{ true \}_**

&emsp;&emsp;&emsp;**・ false: runs the assignment *t.a\_e = e* 和 *t.a\_v = v*, persistently stores *t* ，at last returns *\{ true \}***

&emsp;&emsp;**• false, returns  _\{ false, t.p\_e \}_**

&emsp;**If an Acceptor receives multiple _A(e,v)_ requests simultaneously, the Acceptor guarantees that each _A(e,v)_ request is processed strictly serially, independently and atomicly**

For an outside caller, one Acceptor has three state as follows:

1. Unknown: network unreachable, or timed out;
2. In service: requests can be processed and a valid value can be returned;
3. Unavailable: service not online or has been **dead**.

When an Acceptor is damaged permanently and irreversibly, such as storage file or disk where state table *t* lies has been damaged unrecoverably, or the file has been lost, we call this Acceptor has died. **A dead Acceptor will be at a dead state and never brought back**.

At the same time, each Acceptor must satisfy following constraints:

&emsp;&emsp;**Constraint 0.2**&emsp;**The processing time that each Acceptor takes on service requests ( _P(e)_ or _A(e,v)_ ) is atomically exclusive**,

#### 2.4 Proposer Definition

```lua
proposer_propose(v, e) -> {chosen_flag, help_chosen_flag, chosen_v, next_e}
```

```lua
local majorityN = (len(config0) + 2) / 2
local function proposer_propose(v, e)
    local p_next_e, p_ok_list, max_a_v = send_prepare_to_majority(e)
    if len(p_ok_list) >= majorityN then
        local accept_v = v
        local help_flag = false
        if max_a_v ~= nil then  
            accept_v = max_a_v
            help_flag = true
        end
        local a_ok_list, a_next_e = send_accept_to(p_ok_list, e, accept_v)
        if len(a_ok_list) >= majorityN then
            return {true, help_flag, accept_v, nil}
        else
            return {false, false, nil, a_next_e or p_next_e}
        end
    else
        return {false, false, nil, p_next_e}
	end
end
```

The *majorityN* is defined as follows:

​	For a non-empty set 𝐀 with 𝑥 members (each member differs from each other in 𝐀), there exists a positive integer 𝑦 that satisfies following condition:

​		For any sub-set of 𝐀 with 𝑦 members, there must been a non-empty intersection between them

​	The minimal possible value of 𝑦 of 𝐀 is defined as *majorityN*.

For example, a set with 3 members and *y* could be 2 or 3, then the minimal value of *y* that *majorityN* is 2. More examples are as follows:

・when *x=1,y=1*, *majorityN=1*;

・when *x=2,y=2*, *majorityN=2*;

・when *x=3,y=2,3*, *majorityN=2*;

・when *x=4,y=3,4*, *majorityN=3*;

・when *x=5,y=3,4,5*, *majorityN=3*;

・when *x=6,y=4,5,6*, *majorityN=4*;

**… …**

```LUA
send_prepare_to_majority(e) -> p_next_e, p_ok_list, max_a_v
```

Function *send\_prepare\_to\_majority(e)* sends _Prepare(e)_ request to greater or equal to *majorityN*  (referred to as 'more than half') Acceptors in Paxos configuration, the returned value is defined as:

* *p\_ok\_list* is the Acceptor list that have returned successful result (*true*) to *Prepare(e)* request;
* *max\_a\_v* is the *t.a\_v* value that has the **greatest non-zero *t.a\_e* returned value** among all successful *Prepare(e)* returned values, if not exist, *nil* is returned.
* *p\_next\_e* is a suggested index value used to optimize Proposer's performance. In common its value is the greatest *t.p\_e* value in all *Prepare(e)* returned values plus one, if not exist, *nil* is returned. This value doesn't influence the correctness of Paxos, but might influence the performance.

```lua
send_accept_to(p_ok_list, e, v) -> a_ok_list, a_next_e
```

Function *send\_accept\_to(p\_ok\_list, e, v)*  sends _Accept(e,v)_  to each Acceptor that has successful done Prepare (every Acceptor in parameter *p\_ok\_list*), the returned value is defined as:

* *a\_ok\_list* is the list of Acceptors from set *p\_ok\_list* that have returned successful results to request *Accept(e,v)*;
* *a\_next\_e* is a recommended value for optimizing Proposers, normally set as the greatest *t.p\_e* value from all *Accept(e,v)* returned values plus one, if not exist, *nil* is returned. Its value does not affect the correctness of Paxos algorithm, but may affect the running performance of Paxos algorithm.

```lua
proposer_propose(v, e) -> {chosen_flag, help_chosen_flag, chosen_v, next_e}
```

**Each call of function *proposer\_propose(v, e)* should be regarded as the creation, run and exit of  an unique Proposer, which means a complete life cycle of a Proposer.**

Function *proposer\_propose(v, e)* starts a proposal with value *v*, in which the parameter *e* is a constant, positive integer with arbitrary precision and default value 1, used for optimization of Proposers. The returned values of this function are defined as follows:

* *chosen\_flag*: a boolean value, if true it means a proposal has been *chosen*, otherwise it means current Proposer thinks this proposal has failed (if returned values from some A operations are lost, even though the proposal has been chosen, Proposer will still determine that this proposal has failed)
* *help\_chosen\_flag*: a boolean value, only makes sense when the returned value *chosen\_flag* is true
  * if *chosen\_flag* is true and *help\_chosen\_flag* is true: the final chosen value *chosen\_v* is the value that another Proposer proposed earlier. This means current Proposer has helped another Proposer choose a proposal value *chosen\_v*, instead of choosing the proposal value *v* from parameter.
  * if *chosen\_flag* is true and *help\_chosen\_flag* is false: the chosen proposal value *chosen\_v* is the proposal *v* from parameters, which means this chosen proposal is the proposal from the Proposer itself who has called the function
* *chosen\_v* is an binary string with arbitrary size, that means the chosen value, if not exist, *nil* is returned;
* *next\_e* is a recommended value *e* used for Proposer's optimization, mainly used in the iteration calls of function *proposer\_propose*, usage shown as follows:


```lua
local function must_chosen(v)
    local e = 1
    while true do
        local chosen_flag, help_chosen_flag, chosen_v, next_e =
            proposer_propose(v,e)
        if chosen_flag then
            return help_chosen_flag, chosen_v
        end
        e = next_e		
    end
end
```

### 3 Geometry Meanings and Proofs

#### 3.1 Definition

![paxos_timeline_new_proof_2](img/paxos_timeline_new_proof_2.jpg)

![paxos_timeline_new_proof_3](img/paxos_timeline_new_proof_3.jpg)

Figure 2 shows how Figure 3 is created. Figure 3 shows a **distributed time-line diagram** (abbr. time-line diagram) of a Paxos instance in which there are five Acceptors. Given that at *t = 0*, all Acceptors have finished initiation and ready to serve Prepare and Accept requests from all Proposers.

In the time-line diagram, the horizontal line on the top is the base-line of the system at  *t = 0*, five following vertical lines are **time-line** for each Acceptors. Time scales are equal in all time-lines, which means five crossover points of any horizontal line *t = 𝑥* and vertical time-lines represent the same corresponding point on Acceptor time-line at *t = 𝑥*.

For the convenience of subsequent descriptions, here are some term definitions.

**P** means Prepare operation, **A** means Accept operation. **Unless specified, P or A all means Prepare and Accept operations with successful response in the following sections**

**P Noise** refers to P operations that Proposer has done successful *P(e)* to Acceptors by calling *send\_prepare\_to\_majority(e)* function, but doesn't do successful *A(e, v)* using this *e* value later (the reason may vary, e.g. numbers of successful P responses are less than *majorityN*, or Proposer goes down temporarily, or subsequent A requests are all lost in the network, or subsequent A requests all get non-successful responses). For example, in Figure 3, there are three P Noise 4, 5, 6 at right-bottom area .

P Noises that have greater or equal to *majorityN* successful P operations are called **Majority P Noise**, the others are called **Minority P Noise**

For example at the right-bottom area of Figure 3, P Noise 6 is a Minority P Noise, which only contains one successful P operation. P Noise 5 is a Minority P Noise which contains two successful P operation. P Noise 4 is a Majority P Noise which contains three successful P operations.

**Unchosen PA Event** (Unchosen PA) refers to: Proposers call function _send\_prepare\_to\_majority(e)_ to request _P(e)_ to Acceptors, in which greater or equal to *majorityN* P are successful, then send _A(e, v)_ requests to these successful Acceptors, but with only less than _majorityN_ _A(e,v)_ operations successfully done. The whole set of these P, A operations are called an Unchosen PA event. For example, at the left-bottom area in Figure 3, P operations set P3 and A operations set A3 commonly consist of a Unchosen PA event.

It's easy to prove by contradiction that in shadow area Sp3, S3 and Sa3, there are no such P, A operations except P3, A3 (because if there are, subsequent A3 operation must have failed, which is contradictive to what is known).

**Chosen PA** can be divided to **Help-Chosen PA** and **Self-Chosen PA**

* Help-Chosen PA refers to all successful P operations and A operations related to the call of function *proposer\_propose*  that has returned *chosen\_flag* set to true and *help\_chosen\_flag* set to true, 
* Self-Chosen PA refers to all successful P operations and A operations related to the call of function *proposer\_propose* that has returned *chosen\_flag* as true and *help\_chosen\_flag* as false.

For example P1 and A1 consist of a Chosen PA at the left top area of Figure 3.

It's easy to prove by contradiction that inside shadow area Sp1, S1 and Sa1, there are no other possible sucessful P, A operations except P1 and A1 (because if there are, there must be failures in subsequent A1 operations, which is contradictive to what is known).

Unchosen PA and Chosen PA are referred to as **PA**.

In addition, because time used in network transport must be greater than zero, therefore area S1, S2 and S3 have heights greater than zero.

#### 3.2 Theorems

Assume there is a God role 𝓖, who knows the past, the present and the future. Time makes no sense to 𝓖. **𝓖 only concerns all past, present and future PAs in a Paxos instance**. Here are definitions for some symbols and conventions.

One PA contains two kinds of operations *P(e)* and *A(e, v)*, parameter *e* must be the same in all P and A operations in one PA. We refer to this *e* as the **e value of this PA**, similarly we refer to *v* in *A(e,v)* as the **v value of this PA**, also known as the **proposed value of the proposal** of PA.

One PA could be written as ***PA(e)***, in which e means the e value of corresponding PA. One *PA(e)* may be Chosen PA or Unchosen PA. To distinguish between them, symbols **U** and **C** are defined. **U** refers to as **U**nchosen PA, **C** refers to **C**hosen PA.

Because 𝓖 is all-knowing, therefore a table called **𝓖 Table** can be drawn as follows. The table shows all past, present and future PAs in one Paxos instance.

|   e   |    PA     |     A     |
| :---: | :-------: | :-------: |
| … ... |    U*     |   … ...   |
|  e₀   |     C     | (e₀ , v₀) |
|  e₁   |  U\|\|C   | (e₁ , v₁) |
|  e₂   |  U\|\|C   | (e₂ , v₂) |
|  e₃   |  U\|\|C   | (e₃ , v₃) |
| … ... | (U\|\|C)* |   … ...   |

**𝓖 Table Theorem: For any Paxos instance, there must exist only one corresponding 𝓖 Table, which shows all PAs of current Paxos instance. From the top to the bottom, e value increases monotonically and each has its unique value. All e values satisfy condition *e > 0*. The far right column lists all successful A operations' parameters in the instance, which equally lists all possible valid *(a_e, a_v)* values of a successful returned value of P opration **

**𝓖 Table Theorem 0.1**: For any Paxos instance, there must exist only one 𝓖 Table, which lists all PAs from current Paxos instance. From the top to the bottom e value increases monotonically and each has its unique value, all e values satisfy condition *e > 0*

**Proof** &emsp;First, according to definitions of Acceptor and Proposer, e value of one PA must be greater than zero. Second, if two different PAs that have the same e values, they must have received responses to *P(e)* from the majority of Acceptors, these two majority of Acceptors must have intersection, in which Acceptors must have responded to *P(e)* requests that have the same e values, this is contradictive with the definition of Acceptor service. That is, in one Paxos instance all PAs must have unique e values, therefore 𝓖 Table Theorem 0.1 is proven.

**𝓖 Table Theorem 0.2** &emsp;**The far right colum of 𝓖 Table lists all successful A operations' parameters** 

**Proof**&emsp;A conclusion 'Two different PAs must have different e values' can be drawn according to 𝓖 Table theorem 0.1, and parameters *(e, v)* from all A operations of one PA must be equal. Therefore 𝓖 Table Theorem 0.2 is proven.

**𝓖 Table Theorem 0.3**&emsp;**The far right colum of 𝓖 Table lists all posible *(a_e, a_v)* values from successful P operation responses**

**Proof**&emsp;According to the definition of Acceptor, the response from a successful P must satisfy *a_e >= 0*. When *a_e == 0*, *(a_e, a_v)* is the invalid initiated value. When *a_e > 0*, only successful A operations change *(a_e, a_v)* values in Acceptor state table, thus *(a_e, a_v)* must be equal to the parameter *(e, v)* of a successful A operation. In addition with 𝓖 Table Theorem 0.2, 𝓖 Table Theorem 0.3 is proven.

Because PA must be either *U*nchosen or *C*hosen, then **𝓖 Table must have a *chosen PA* with the smallest e value**. We define this e value as *e₀*, the event as *PA(e₀)*, the v value from A request as v₀, in total as *A(e₀ , v₀)*.

Similarly, definitions shown successively as follows:

* Event *PA(e₁)* is the PA event with smallest e value from PA events that have e values greater than e₀, of which A operation defined as *A(e₁ , v₁)*
* Event *PA(e₂)* is the PA event with second smallest e value from PA events that have e values greater than e₀, of which A operation defined as  *A(e₂ , v₂)*
* Event *PA(e₃)* is the PA event with second smallest e value from PA events that have e values greater than e₀, of which A operation defined as  *A(e₃ , v₃)*
* … ...
* Definitions go on and so on

Suppose the state table of one Acceptor is *{p_e, a_e, a_v}*, according to the definition of Acceptor service we can draw the conclusion: only successful P requests change the *p_e* value of the Acceptor, only successful A requests change *(a_e, a_v)* value of the Acceptor. Therefore we can obtain the **Theorem of Acceptor's Monotonicity**

**Theorem of Acceptor's Monotonicity**&emsp;**For a specific Acceptor of a Paxos instance, which has state table *{p_e, a_e, a_v}*. The *p_e* value in it increases monotonically along with time, *a_e* value increases monotonically along with time as well, they always satisfy *a_e <= p_e***.

It's easy to prove according to the definition of Acceptor service.

#### 3.3 Mathematical Proof

This section provides a pure mathematical proof, you can jump over to the next section **geometric proof** if not interested.

**Paxos Theorem**&emsp;**In a running Paxos instance, suppose the chosen PA event *PA(e₀)* with the smallest e value has chosen proposal value *v₀* , then all the other chosen PA events must be help-chosen PA events, and the chosen proposal value must be equal to *v₀***

**Proof**

![paxos_timeline_new_proof_6](img/paxos_timeline_new_proof_6.jpg)

All referenced P, A operations in the following proof are successful P, A operations if not specifically stated.

Core idea of this proof is

&emsp;**Using events that have happened in 𝓖 Table to deduce necessary constrained relation between these events**

**Proof 0.1**

For any successful P request *P(e)*

![paxos_timeline_new_proof_14](img/paxos_timeline_new_proof_14.jpg)

Successful returned value of P is defined as *{true, a_e, a_v}*, the newest state table of Acceptor before serving *P(e)* is defined as *{p_e, a_e, a_v}*. According to the definition of Acceptor service, we can obtain *e > p_e* and *a_e ≤ p_e*, that is *e > a_e* must be true, Therefore we have conclusion **0.1.0**

&emsp;**Conclusion 0.1.0**&emsp;**Successful returned value of *P(e)* must satisfy *a_e < e***

Then for event *PA(e₁)* we have conclusion:

&emsp;**Conclusion 0.1.1**&emsp;**All successful returned value of P in *PA(e₁)* must satisfy *a_e < e₁***

Together with the definition of Proposer, we can have conclusion **0.1.2**

&emsp;**Conclusion 0.1.2**&emsp;**If a returned value of P satisfy *a_e == e₀*, then *a_v == v₀***

Because *PA(e₀)* is a chosen PA event，the Acceptors set of all its successful *A(e₀ , v₀)* operations must have a non-empty intersection with the Acceptors set of all successful *P(e₁)* operations inside *PA(e₁)*. Inside this intersection set the returned values of *P(e₁)* operations of Acceptors must satisfy *a_e ≥ e₀*, together with conclusion 0.1.1

![paxos_timeline_new_proof_11](img/paxos_timeline_new_proof_11.jpg)

We have

&emsp;**Conclusion 0.1.3**&emsp;**At least one returned value of *PA(e₁)* of all successful P operations satisfy *e₀ ≤ a_e < e₁***

Because v value of A operation of *PA(e₁)* event is the *a_v* value of successful P returned value that has the greatest non-zero *a_e* value among all successful *P(e₁)* returned values. When such *a_v* doesn't exist, a new proposal value v can be proposed. According to Acceptor monotonicity theorem, 𝓖 Table Theorem 0.3, conclusion 0.1.1, 0.1.2, 0.1.3 and Proposer definition

![paxos_timeline_new_proof_16](img/paxos_timeline_new_proof_16.jpg)

We have

&emsp;**Conclusion 0.1.4**&emsp;**v value of A operation in *PA(e₁)* event must be v₀**

**Proof 0.2**

We can draw a conclusion based on Conclusion 0.1.0

&emsp;**Conclusion 0.2.1**&emsp;**All P successful returned value of *PA(e₂)* event must satisfy *a_e < e₂***

According to Proof 0.1, Conclusion **0.1.4** and Proposer definition, we can draw a conclusion

&emsp;**Conclusion 0.2.2**&emsp;**If a P returned value satisfy *a_e == e₀ || a_e == e₁*，then we must have *a_v == v₀***

Because *PA(e₀)* is a chosen PA,  the Acceptor set of all its successful *A(e₀ , v₀)* operations must have made a intersection set with the Acceptor set of all successful *P(e₂)* operations, returned values to *P(e₂)* of Acceptors in this set must satisfy *a_e ≥ e₀*, in addition with conclusion 0.2.1

![paxos_timeline_new_proof_13](img/paxos_timeline_new_proof_13.jpg)

We have

&emsp;**Conclusion 0.2.3**&emsp;**At least one returned value of all successful *PA(e₂)* satisfy *e₀ ≤ a_e < e₂***

Because the v value of A operation in *PA(e₂)* event is the *a_v* value of successful returned value of  *P(e₂)* that has the greatest non-zero *a_e*, if no such *a_v* a new value can be proposed. According to Acceptor monotonicity Theorem, 𝓖 Table Theorem 0.3, Conclusion 0.2.1, 0.2.2, 0.2.3 and Proposer definition

![paxos_timeline_new_proof_17](img/paxos_timeline_new_proof_17.jpg)

We have

&emsp;**Conclusion 0.2.4**&emsp;**v value of A operation in *PA(e₂)* event must be v₀**

**Proof 0.3**

Using method in Proof 0.2, prove that *PA(e)*  that has e value greater than e₂ in order from small to large satisfy

&emsp;**v value of A operation in *PA(e)* must be v₀**

Which leads to the conclusion 'Paxos algorighm is correct' according to the second mathematical induction

#### 3.4 Geometry Proof

**Constraints Theorem of Time-line Diagram** There must exist a time-line diagram between one chosen PA and one PA, in which the order of all Acceptors from left to right satisfy:

&emsp;**All Acceptors of successful A request of Chosen PA in the time-line diagram are adjacent **

&emsp;**All Acceptors of successful P request of PA in the time-line diagram are adjacent**

![paxos_timeline_new_proof_7](img/paxos_timeline_new_proof_7.jpg)

**Proof**

Set A∪C is the set where all Acceptors  successful A operations

Set C∪B is the set of all Acceptors of all successful P operations of the second PA

Set C is actually the intersection of two majority Acceptor, therefore C must be non-empty, A and B may be empty.

Therefore, there must be at least one time-line diagram shown that satisfy, so the Theorem is proven.

**Paxos Theorem**&emsp;**In a running Paxos instance, suppose the chosen PA with smallest e value *PA(e₀)* has chosen proposal value *v₀*, then all the other chosen PA must be help-chosen PA, and the proposal value must be equal to *v₀***

**Proof**

![paxos_timeline_new_proof_8](img/paxos_timeline_new_proof_8.jpg)

Because

* Set C∪B Acceptors to *P(e₁)* returned value must satisfy *a_e < e₁* (according to conclusion 0.1.0)
* Set C Acceptor to *P(e₁)* returned value satisfy  *e₀ ≤ a_e < e₁*
* Set C must be non-empty

Therefore, according to the above discussion and Acceptor monotonicity Theorem, 𝓖 Table Theorem 0.3 and Proposer definition, we have

&emsp;***PA(e₁)* A operation v value must be v₀**

![paxos_timeline_new_proof_9](img/paxos_timeline_new_proof_9.jpg)

Because

* Set C∪B Acceptor to P(e₂) returned value must satisfy *a_e < e₂* (according to conclusion 0.1.0)
* Set C Acceptor to P(e₂) returned value must satisfy  *e₀ ≤ a_e < e₂*
* Set C must be non-empty，and *PA(e₁)* A operation v value must be equal to v₀

Therefore, according to the above discussion and Acceptor monotonicity Theorem, 𝓖 Table Theorem 0.3 and Proposer defintion

&emsp;***PA(e₂)* A operation v value must be v₀**

Using the same method, we can prove *PA(e)* with  e values greater than e₂ satisfy

&emsp;***PA(e)* A operation v value must be v₀**

To sum up, according to the second mathematical induction, Paxos has been proved

At last we can have the geometry meaning: drawing random lines on the distributed timeline diagram, as long the basic constrain is satisfied, no matter how the line is drawn, the final result must be 'chosen values in all chosen PA must be the same'. Using the fact that 'any two majority of Acceptors must intersect', restricting the monotonicity of e value then sequencing all PA globally, the system then goes forward on the virtual timeline of e. Therefore as long as having defined the start state of system and constraint conditions of transition of two states (mathematical induction), we can implement the design goal. When researching Paxos, it is better to make 𝓖 as the observer instead of some specific role in the system (Acceptor or Proposer), because it makes easy to eliminate 'unknown' states of the system, which leads to the core area of connections of events in the system.

#### 3.5 Others

In the previous proof, we haven't considered the situation in common asynchronous communication system where 'message duplication', why is it?

First, in real system implementation, TCP is used to avoid message duplication, which is a very common usage in practice. For example, a lot of RPC frameworks are based on TCP, in addition normally we assume TCP satisfy its commitments, otherwise it can be regarded as Byzantine faults.

Second, using the simplified model lets us concern the core of Paxos algorithm itself, instead of other trivial matters. In fact, in the proof of Paxos algorithm, 'message duplication' is just a special case of 'message without duplication'.

Next, we will extend the previous proof to the situation where 'message duplication'

**Proof**&emsp;Suppose during a run of function `proposer_propose`, 'message duplication' happened in P request, this actually has no effect on the correctness of Paxos algorithm, because

&emsp;**Acceptor If an Acceptor receives multiple P requests with the same e value, then there there must be at most one P request that succeeds**

Whichever response of P operation that Proposer receives, those P operations without responses may all be regarded as P Noises sent by the other one or more Proposers.

Using similar method, we have following discussions:

&emsp;**1.** When a Proposer enters Prepare phase

&emsp;**1.1.** Suppose messages are not duplicated, P request fails, then message duplication doesn't affect the system at all

&emsp;**1.2.** Suppose messages are not duplicated, P request succeeds

&emsp;**1.2.1.** Suppose a Proposer receives a successful response of P request, then message duplication doesn't affect the system at all

&emsp;**1.2.2.** If a Proposer receives a failed response of P request, message duplication only affects whether the Proposer is able to recognize if it has acquired successful responses to its P request from the majority of Acceptors. In this case this only affects whether this Proposer can continue the Accept phase of the algorithm, but doesn't affect the correctness of Paxos algorithm.

&emsp;**2.** Suppose a Proposer has successfully enters Accept phase, then

&emsp;**2.1.** Suppose message not duplicated, A requests fails, message duplication doesn't affect the system at all in this case

&emsp;**2.2.** Suppose message not duplicated, A request succeeds

&emsp;**2.2.1.** If a Proposer receives a successful response of A request, message duplication doesn't affect the system at all in this case

&emsp;**2.2.2.** Due to at least one successful P request inserted in multiple identical A requests, and Proposer has received failed response to A request, message duplication only affects whether the Proposer is able to recognize timely and correctly if `proposer_propose` it has sent has chosen a value successfully (no matter help-chosen or self-chosen), but doesn't affect the correctness of Paxos algorithm

End of proof.

### 4 Summary

When learning and proving Paxos, problem would become extremely complicated and uncontrollable if only concerning relations of each event on time-domain in a distributed system. Because in this case, the system is very close to the real world where each individual affects each other, which is very complicated. On the contrary, mathematic is simple. Therefore, according to the premise 'no Byzantine fault in the system', we can make deduction using definitions of Acceptor and Proposer services, and transform the problem 'time-domain issue in distributed system' into the issue in ' monotonical integer domain' -- e domain (e value in PA). This simplifies the description, understanding and proof.

Here are some common wrong understandings of Paxos and their rectification:

* Paxos is a distributed **consistency algorithm**, but not a election algorithm. One paxos instance only runs among fixed numbers of computers and commits one unique value. In production environment a election algorithm has to implement the application, release, timed-out release and listening of **distributed lease lock**. Of course it's easy to implement distributed lock based on the implementation of **replicated state machine** based on Paxos.
* There is an opinion on introduction and implementation of Paxos: e values of different Proposers must not have intersection, for example e values are set by the modular value of Proposers number. In fact this constraint is not necessary and useless in reality at all. Because even different Proposers use the same e value to make P, A requests, it doesn't affect the correctness of the algorithm. Moreover, this method cannot guarantee that no overmuch conflicts happen during Paxos running (common solution is to elect or specify a Master in Proposers).
* Number of Acceptors doesn't have to be odd, it works as long as it's greater than 1. The reason the number is always odd in production environment is that, odd numbers of Acceptors brings best benefits compared to cost in total. For example, if the number is 3, the MajoritN is 2, the system tolerates one Acceptor down. If the number is 4, the MajorityN is 3, the system still tolerates one Acceptor down. But if the number increases to 5, the system tolerates two Acceptor down even though the MajorityN is still 3.

One Paxos instance has very limited functions, even could be useless at all in reality. The real powerful function is to create **replicated state machines, abbr. RSM** based on it, **it combines infinite Paxos instances together in order, abstract them into one strong consistent, fault-tolerate, high-available infinite distributed instruction paper tape**. Several state machines (processors) with the same initial states runs through instructions from start to end in order. Suppose each instruction is mathematically determined, then it is sure that all internal states of processors must be identical after they have run the same instruction at the same location. This is why this model is referred to as 'Replicated State Machine'. In it's core, RSM consists of a strong-consistent, fault-tolerant, highly-available distributed state machine based on the infinite distributed instruction paper tape with the same properties.

Another suggestion: In real system design, all optimization, customization and extending of Paxos should be done by **adding new constraints based on Paxos, instead of modifying Paxos itself**, unless the new modified algorithm can be proved strictly and mathematically by designers.

If you want to know more applications of Paxos, welcome to [Official Website](https://turingcell.org) of open-source project TuringCell.

### 5 The philosophical meaning behind mathematical proof

> The most incomprehensible thing about the world is that it is at all comprehensible. -- Albert Einstein
>

‘comprehensible’ implies that logically the world is not self- contradictory, which is, we're able to prove a topic true or false using rational mind such as logical reasoning or contradict.

&emsp;**Godel's incomplete first theorem**&emsp;**An axiom system containing Piano arithmetic, if it is self-negotiating, then it is incomplete, that is, it can be constructed in this axiom system can not be proved true and false proposition**

&emsp;**Godel incomplete second theorem**&emsp;**Axiom system containing Piano arithmetic, if it is self-negotiating, then its self-harmony can not be proved true or false only in this axiom system**

For example, Goldbach's guess is likely to be in the collection of incomplete propositions in primary number theory (i.e., it is likely that new axioms need to be expanded into primary number theory to be proved, and from this we can also see the significance of Goldbach's conjecture, as it represents a further step in the depth of human exploration into the unknown realm of nature)

&emsp;**Goldbach‘s Conjecture**&emsp;Any even number greater than 2 can be represented as the sum of two prime numbers

So far, the self-harmony of primary number theory has not been fundamentally proved, we can only choose to believe, because, first, it conforms to the intuitive abstract feeling of human beings, second, in the practical application of primary number theory and derivative theory, has not found examples of self-engagement, and third, if this is also a reason, it is that we really have no choice but to do so.

Therefore to sum up, we can assume:

&emsp;**P0.1**&emsp;**The world we live in must be self-consistent**

&emsp;**P0.2**&emsp;**In a distributed system where asynchronous communication can take place between members, if the behavioral definitions of all its members are reasonable, then all past, present, and future events in the system and the logical connections between them must be self-negotiating**

For example, in real life detectives in the case analysis, reasoning is based on P0.1. That is, "existence is reasonable", where "reasonable" refers to rational logic that does not create logical self-contradictions (not "correct behavior", "ethical behavior", etc.) in accordance with human subjective values.

In this paper, our proof of the correctness of the Paxos algorithm is based on P0.2 -- a new conclusion derived from P0.1.

### 6 Community, Support and Cooperation

Welcome to [TuringCell Community](https://github.com/turingcell/join-community)！

Your participation, support and feedback are important to this open-source project! Interchange and collision of ideas, openness, tolerance, equality and freedom are the real charm of open-source! Become one of our community and let's create the next exciting distributed project!

You can choose to support this project either by joining email list, WeChat group, applying for Github TuringCell org, sharing, issuing, Star/Watch/Follow or donation.

In addition, any other forms of cooperations are welcome as long as benefiting the open-source project. Please contact us.

### 7 Donation

Very grateful to your [Generous Donation](https://github.com/turingcell/donate)！

### 8 Update History

```
v0.01 2015.3 
    Sen Han (韩森) <00hnes@gmail.com>

v1.0  2017.12-2018.2 
    Sen Han (韩森) <00hnes@gmail.com>

v1.1  2020.5 
    Sen Han (韩森) <00hnes@gmail.com>
```

### 9 Copyright and License

Author: Sen Han (韩森) <00hnes@gmail.com>

Translator: Yicheng Liu (刘易承)  liuyicheng01@outlook.com

Website: https://github.com/turingcell/paxos-made-easy

License: This work is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/), except the picture of the [TuringCell Logo](https://github.com/turingcell/logo) which is under the [Creative Commons Attribution-NoDerivatives 4.0 International License](http://creativecommons.org/licenses/by-nd/4.0/).









