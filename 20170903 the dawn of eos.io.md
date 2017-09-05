# EOS.IO의 여명

![](https://steemitimages.com/DQmPgU9hugYig9dCJWYfpfDkZLA5Tmshk5qhTXg7rcfzuSq/image.png)

[EOS.IO 기술 백서](https://github.com/EOSIO/Documentation/blob/master/TechnicalWhitePaper.md)에서 우리는 EOS.IO 소프트웨어를 블록체인 컴퓨팅의 새로운 시대를 여는 기술로 제안하였습니다. EOS.IO 개발 팀은 무더운 여름 누구보다 열심히 일하였습니다. 여름은 끝이났고, EOS.IO 소프트웨어의 개발은 예정보다 빠르게 진행되었습니다. 이제 분산 네트워크 환경에서 동작합니다. EOS.IO 소프트웨어 개발에 대한 짜릿한 소식들을 여러분께 선사드립니다. 마지막까지 읽어주세요!

# 성능 증명

EOS.IO 소프트웨어를 분산 네트워크에서 구동하여 성능 측정을 하였습니다. 내부 테스트에 따르면 현재 다중 노드 네트워크에서 싱글 쓰레드(1개의 CPU 코어를 사용)로 초당 10,000개의 트랜잭션(TPS)을 처리할 수 있습니다. 만약에 100개의 CPU 코어를 가진 컴퓨터에서 처리한다면 100만 TPS를 달성할 수 있습니다.

## 디자인 개선

개발자들은 우리의 최신 소프트웨어 아키텍처로 서로 통신하는 병렬 응용 프로그램을 쉽게 구축할 수 있을 것입니다.

### 공유 데이터베이스 액세스

복잡한 비동기 통신 없이 다른 응용 프로그램의 데이터베이스 상태(state)를 읽을 수 있게 되었습니다. 각 트랜잭션이 읽기 또는 쓰기 접근 권한이 필요한 범위(scope; 데이터 범위)를 선언할 수 있게하여 병렬로 실행하면서 가능하게 하였습니다. 블록 생산자들은 데이터 충돌이 일어나지 않도록 트랜잭션을 스케쥴링합니다.

### 애플리케이션 데이터의 사용자 로컬 저장소

계정 간 읽기 엑세스를 지원할 뿐만 아니라, 애플리케이션은 다른 계정에 데이터를 저장할 수 있습니다. 이제 통화 스마트 컨트랙트에서 사용자들의 잔액 정보를 컨트랙트 내부에 저장하는 것 대신에 각각의 사용자 계정에 저장할 수 있습니다. 앨리스가 밥에게 이체할 때에 앨리스와 밥의 읽기/쓰기 권한만 필요하게 되며 통화 스마트 컨트랙트의 데이터 범위(scope)는 영향을 주지 않습니다. 이로 인해 많은 종류의 애플리케이션은 쉽게 병렬화 가능하며 단일 스레드 처리량 한도를 초과하는 통화 전송을 처리할 수 있습니다. 우리가 알고있는 한, 다른 블록체인은 병렬 소프트웨어 아키텍처를 개발하기 위한 이러한 확장성 있고 쉬운 접근법을 제공하지 않습니다.

### 인라인 메시지 전달

다른 애플리케이션으로 메시지를 보낼 때 수신 여부와 유효성의 확인이 어느때보다 쉬워집니다. 애플리케이션은 현재 트랜잭션의 마지막 부분에 원하는 만큼 추가 메시지를 만들 수 있습니다. 만들어진 메시지가 동일한 읽기/쓰기 데이터 범위(scope)를 가질 때 할당된 시간에 처리되고, 전송이 보장되거나 혹은 원래대로 되돌릴 수 있습니다.

이 접근법은 다른 플래폼의 동기식(synchronous) 접근 방법과 다릅니다. 반환될 때 까지 멈추는(block) 동기식 메시지 전달은 예상치 못한 재진입(reentrancy) 가능성이 만들어집니다. 재진입은 개발자가 동기식 호출을 할 때 스마트 컨트랙트가 안정된 상태에 있음을 확신하기 어렵게 만들기 때문에 수많은 버그와 취약점의 원인이 되었습니다. 인라인 메시지 전달을 사용하면 

This approach is different than the synchronous approach used by other platforms. Synchronous message delivery, which blocks execution of the current thread until it returns, creates the potential for unanticipated reentrancy. Reentrancy has been a source of numerous bugs and exploits because it is difficult for developers to ensure their contract is in a consistent state prior to making a synchronous call. With inline message passing, which delays execution until the end of the current transaction handler, developers can dispatch a message and proceed as if it succeeded. If it fails then the entire transaction will be unwound without any harmful side effects. This means your message handlers are never called in an inconsistent state.



### Deferred Message Passing
Sometimes you don’t know if a message is valid or whether there is enough time left on the clock to execute inline with the current transaction. Other times you need to send a message that accesses data outside the scope of the current transaction.  In this situation applications can request the block producers schedule a message to be delivered in the next cycle or a future block. If it is valid then your application may be notified; if it is not, then it will never be scheduled and your application can clean up after a timeout.

### Unlimited Horizontal Scaling
The latest design advancements in the EOS.IO software gives developers high single-machine performance; businesses can scale to a million transactions per second before requiring a more complex asynchronous architecture.

That said, the EOS.IO software will still support asynchronous message passing among groups of applications that do not need to share state. There are many benefits to async message passing (such as trivial cluster support), but those benefits come with the cost of greater development complexity; the EOS.IO software supports this for businesses that require several millions of transactions per second, but offers a streamlined approach for those that don’t.

# Next Generation Network Topology
**The EOS.IO software is designed to empower block producers to provide a high performance decentralized infrastructure as a service.** Application developers need more than a set of block producers aggregating transactions, they need API nodes, seed nodes, database indexes, storage, and hosting.

High performance blockchains demand high performance network architectures with very different requirements from existing blockchains. At a million transactions per second each node is required to achieve 100’s of megabytes per second per connection. This is trivial for large data centers, but inconceivable for home users.

Additionally high performance blockchains consist of heterogeneous nodes running different subsets of the blockchain and will likely prune the transaction history. This is a significant departure from prior blockchain systems where all nodes are identical and have a full history.  

A ***traditional blockchain*** consists of a dynamic set of ***randomly connected nodes*** in a mesh network. They target home users with limited bandwidth and are designed to traverse home routers (NAT) and dynamically add nodes to the network. Our observation is that this architecture is not well suited for high performance blockchain infrastructure.

The ***EOS.IO software*** starts with the assumption that ***all nodes are intentionally connected to each other***. Node operators work together to ensure the network topology is secure, well planned, and efficient. This allows block producers to establish direct (and secure) connections to each other and prevents attackers from scanning the entire network topology looking for nodes to shut down.  

The block producers will host public endpoints which anyone may connect to and subscribe to any subset of transaction data they desire.  This will minimize the bandwidth requirements for full nodes operated by non-block producers.  Nodes that do not want to trust a single block producer may either subscribe to multiple sources or wait for confirmation by ⅔ of the block producers (about 45 seconds). 

The benefit of this architecture is that new nodes can connect and synchronize at very high speeds from high bandwidth infrastructure provided by the block producers. Furthermore, this architecture is designed to facilitate efficient unidirectional streaming rather than less efficient bidirectional protocols. 

**At scale, block producers will be operating a new internet backbone powered by EOS.IO software.** Block producers will be like Tier-1 Internet providers with dedicated fiber optic connections across continents. These producers will operate data centers that Tier-2 subscribers can connect to.  Tier-2 includes anyone looking to run a full or partial node or a large application.  For example, services like block explorers, web wallets,  and crypto-currency exchanges would be Tier-2 subscribers to the block producers. 

We feel this architecture of intentional cooperative network building will enable block producers to offer a quality of service unique in the cryptocurrency industry.  

# The Road Ahead
In September of this year, block.one will be releasing EOS.IO Dawn 1.0 which should be stable enough and well documented enough for anyone to launch their own test network upon which they can build and deploy their applications.  EOS.IO Dawn 1.0 will be the first pre-release of our EOS.IO SDK (Software Development Kit).  

Those who have followed our [EOS.IO Roadmap](https://github.com/EOSIO/Documentation/blob/master/Roadmap.md) will be happy to know that ***we are ahead of schedule***.  Phase 1, The Minimal Viable Testing Environment,  which includes a standalone node, native contracts, virtual machine API, RPC interface, command line tools (eosc), and basic developer documentation is complete. We will be making a tagged release as “EOS.IO Dawn 1.0”.  This phase was scheduled to be complete in Summer 2017 which ends on September 22.  

We have already completed half of **Phase 2, the Minimal Viable Test Network**. This phase is scheduled for completion in Fall 2017 and includes working networking code, virtual machine sandboxing, resource usage and rate limiting, genesis importing, and inter blockchain communication.  At this time we already have functional distributed networks and virtual machine sandboxing.  We are confident that we will complete Phase 2 on schedule.  

EOS.IO Dawn 2.0, the next major pre-release, will come by the end of the year. EOS.IO Dawn 2.0 will include several critical features that are not present in EOS.IO Dawn 1.0 including:

- Resource Rate Limiting (preventing spam / abuse)
- Merkle Tree Generation (for cross chain communication)
- Upgrade Management and Governance
- More robust SDK
- General Infrastructure improvements
- Example Snapshot from ERC20 tokens

The goal of EOS.IO Dawn 2.0 is to be functional enough that one could launch a live blockchain. 

# One More Thing…. 

# EOS.IO Storage! 
For the first time, developers will be able to create and deploy a decentralized application and web interfaces without having to worry about bandwidth and storage costs, or even hosting any servers themselves; this enables a host of new innovative decentralized business models, such as a decentralized YouTube, Soundcloud, or other storage-intensive projects.

In addition to computational bandwidth, **native EOS.IO software-based blockchain token holders will now have access to free cloud storage**, hosting, and download bandwidth via IPFS / HTTPS; this access can be used without consuming or transferring tokens.

To achieve this, block producers will host files via IPFS/HTTPS for users and allow other users to download those files. Storage resources are paid for through blockchain emissions and are rate limited to token holders pro-rata to their holdings; like the EOS.IO bandwidth model, storage does not expend EOS.IO software-based blockchain tokens and per-token storage capacity will increase over time with block producer hardware upgrades.

The EOS.IO software storage solution can also support public hosting for those who don’t have any tokens; more details will be released at upcoming blockchain industry events occurring in Shanghai and London.

# Disclaimer
<i>block.one is a software company and is producing the EOS.IO software as free, open source software. This software may enable those who deploy it to launch a blockchain or decentralized applications with the features described above. block.one will not be launching a public blockchain based on the EOS.IO software. It will be the sole responsibility of third parties and the community and those who wish to become block producers to implement the features and/or provide the services described above as they see fit. block.one does not guarantee that anyone will implement such features or provide such services or that the EOS.IO software will be adopted and deployed in any way.

All statements in this document, other than statements of historical facts, including any statements regarding block.one’s business strategy, plans, prospects, developments and objectives are forward looking statements. These statements are only predictions and reflect block.one’s current beliefs and expectations with respect to future events and are based on assumptions and are subject to risk, uncertainties and change at any time. We operate in a rapidly changing environment. New risks emerge from time to time. Given these risks and uncertainties, you are cautioned not to rely on these forward-looking statements. Actual results, performance or events may differ materially from those contained in the forward-looking statements. Some of the factors that could cause actual results, performance or events to differ materially from the forward-looking statements contained herein include, without limitation: market volatility; continued availability of capital, financing and personnel; product acceptance; the commercial success of any new products or technologies; competition; government regulation and laws; and general economic, market or business conditions. Any forward-looking statement made by block.one speaks only as of the date on which it is made and block.one is under no obligation to, and expressly disclaims any obligation to, update or alter its forward-looking statements, whether as a result of new information, subsequent events or otherwise.</i>