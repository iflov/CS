---
title: 네트워크 기초
created: 2024-12-31
updated: 2024-12-31
tags:
  - CS
  - Network
  - Topology
  - Node
  - Link
category: Network
status: 완료
---

# 🌐 네트워크 기초 (Network Basics)

## 📌 개요

네트워크는 **노드(Node)**와 **링크(Link)**로 구성된 데이터 통신 시스템입니다. 여러 컴퓨터나 장치들이 서로 연결되어 정보를 주고받을 수 있게 해주는 기술적 기반입니다.

> [!info] 핵심 개념
> - **노드(Node)**: 네트워크의 연결 지점 (컴퓨터, 라우터, 스위치 등)
> - **링크(Link)**: 노드들을 연결하는 통신 경로
> - **토폴로지(Topology)**: 네트워크의 물리적/논리적 구성 형태

## 📚 목차

1. [네트워크 구성 요소](#네트워크-구성-요소)
2. [네트워크 토폴로지](#네트워크-토폴로지)
3. [실제 구현 예시](#실제-구현-예시)
4. [참고 자료](#참고-자료)

## 네트워크 구성 요소

### 노드 (Node)
- 네트워크상의 **연결 지점** 또는 **종단점**
- 데이터를 송수신할 수 있는 모든 장치
- 예: 컴퓨터, 스마트폰, 라우터, 서버

### 링크 (Link)
- 노드 간의 **물리적/논리적 연결**
- 데이터가 실제로 이동하는 **경로**
- 유선(이더넷, 광섬유) 또는 무선(WiFi, Bluetooth)

## 네트워크 토폴로지

네트워크 토폴로지는 노드와 링크의 **배치 구조**를 의미합니다.

### 1. 🔄 Ring (링형)
- 각 노드가 양 옆 노드와 연결되어 **원형** 구조
- 데이터가 한 방향으로 순환
- 장점: 구조가 단순, 설치 비용 저렴
- 단점: 하나의 노드 고장 시 전체 네트워크 마비

### 2. 🕸️ Mesh (망형)
- 노드들이 **다중 경로**로 연결
- 높은 신뢰성과 중복성 제공
- 장점: 경로 다양성, 장애 허용
- 단점: 복잡도 높음, 비용 증가

### 3. ⭐ Star (성형)
- 중앙 노드를 중심으로 모든 노드가 연결
- 현재 가장 **널리 사용**되는 구조
- 장점: 관리 용이, 노드 추가/제거 간편
- 단점: 중앙 노드 장애 시 전체 마비

### 4. 🔗 Fully Connected (완전 연결형)
- 모든 노드가 서로 **직접 연결**
- 최대의 연결성과 신뢰성
- 장점: 최단 경로, 높은 가용성
- 단점: 비용 매우 높음, 확장성 제한

### 5. ➖ Line (선형)
- 노드들이 **일직선**으로 연결
- 가장 단순한 구조
- 장점: 구현 간단, 비용 최소
- 단점: 확장성 부족, 중간 노드 장애에 취약

### 6. 🌳 Tree (트리형)
- **계층적 구조**로 노드 연결
- 루트 노드에서 분기
- 장점: 확장 용이, 계층적 관리
- 단점: 루트 노드 의존성 높음

### 7. 🚌 Bus (버스형)
- 하나의 **공통 통신선**에 모든 노드 연결
- 과거 이더넷의 기본 구조
- 장점: 설치 간단, 비용 저렴
- 단점: 충돌 발생 가능, 보안 취약

## 실제 구현 예시

### TypeScript로 네트워크 토폴로지 모델링

```typescript
// 네트워크 노드 인터페이스
interface NetworkNode {
  id: string;
  type: 'computer' | 'router' | 'switch' | 'server';
  ipAddress: string;
  connections: NetworkNode[];
}

// 네트워크 링크 인터페이스
interface NetworkLink {
  from: NetworkNode;
  to: NetworkNode;
  bandwidth: number; // Mbps
  latency: number;   // ms
  type: 'wired' | 'wireless';
}

// 네트워크 토폴로지 클래스
class NetworkTopology {
  private nodes: Map<string, NetworkNode>;
  private links: NetworkLink[];

  constructor() {
    this.nodes = new Map();
    this.links = [];
  }

  // 노드 추가
  addNode(node: NetworkNode): void {
    this.nodes.set(node.id, node);
  }

  // 링크 추가
  addLink(link: NetworkLink): void {
    this.links.push(link);
    // 양방향 연결 설정
    link.from.connections.push(link.to);
    link.to.connections.push(link.from);
  }

  // Star 토폴로지 생성
  createStarTopology(
    centralNode: NetworkNode, 
    peripheralNodes: NetworkNode[]
  ): void {
    peripheralNodes.forEach(node => {
      this.addNode(node);
      this.addLink({
        from: centralNode,
        to: node,
        bandwidth: 1000,
        latency: 1,
        type: 'wired'
      });
    });
  }

  // 네트워크 경로 찾기 (BFS 알고리즘)
  findPath(sourceId: string, targetId: string): NetworkNode[] | null {
    const source = this.nodes.get(sourceId);
    const target = this.nodes.get(targetId);
    
    if (!source || !target) return null;

    const queue: NetworkNode[] = [source];
    const visited = new Set<string>();
    const parent = new Map<string, NetworkNode>();
    
    visited.add(sourceId);

    while (queue.length > 0) {
      const current = queue.shift()!;
      
      if (current.id === targetId) {
        // 경로 재구성
        const path: NetworkNode[] = [];
        let node: NetworkNode | undefined = target;
        
        while (node) {
          path.unshift(node);
          node = parent.get(node.id);
        }
        return path;
      }

      for (const neighbor of current.connections) {
        if (!visited.has(neighbor.id)) {
          visited.add(neighbor.id);
          parent.set(neighbor.id, current);
          queue.push(neighbor);
        }
      }
    }
    
    return null;
  }

  // 네트워크 상태 진단
  diagnoseNetwork(): {
    totalNodes: number;
    totalLinks: number;
    averageConnections: number;
  } {
    const totalNodes = this.nodes.size;
    const totalLinks = this.links.length;
    
    let totalConnections = 0;
    this.nodes.forEach(node => {
      totalConnections += node.connections.length;
    });
    
    return {
      totalNodes,
      totalLinks,
      averageConnections: totalConnections / totalNodes
    };
  }
}

// 사용 예제
const network = new NetworkTopology();

// 중앙 스위치 생성
const centralSwitch: NetworkNode = {
  id: 'switch-001',
  type: 'switch',
  ipAddress: '192.168.1.1',
  connections: []
};

// 컴퓨터 노드들 생성
const computers: NetworkNode[] = Array.from({ length: 5 }, (_, i) => ({
  id: `pc-${i + 1}`,
  type: 'computer',
  ipAddress: `192.168.1.${10 + i}`,
  connections: []
}));

// Star 토폴로지 구성
network.addNode(centralSwitch);
network.createStarTopology(centralSwitch, computers);

// 네트워크 진단
console.log(network.diagnoseNetwork());
// 출력: { totalNodes: 6, totalLinks: 5, averageConnections: 1.67 }
```

## 🎯 핵심 요약

> [!important] 핵심 포인트
> 1. 네트워크는 **노드**와 **링크**의 조합
> 2. 토폴로지 선택은 **비용**, **성능**, **신뢰성**의 균형
> 3. 현대 네트워크는 주로 **Star 토폴로지** 기반
> 4. 각 토폴로지는 고유한 **장단점** 보유

## 참고 자료

### 📖 추가 학습
- [Network Topology - Wikipedia](https://en.wikipedia.org/wiki/Network_topology)
- [Computer Networks - Andrew S. Tanenbaum](https://www.pearson.com/en-us/subject-catalog/p/computer-networks/P200000003179)
- [네트워크 토폴로지 상세 설명](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%ED%86%A0%ED%8F%B4%EB%A1%9C%EC%A7%80)

### 🔗 관련 문서
- [[02_osi-7-layer|OSI 7계층 모델]]
- [[03_tcp-ip-model|TCP/IP 4계층 모델]]
- [[08_network-components|네트워크 구성요소]]
