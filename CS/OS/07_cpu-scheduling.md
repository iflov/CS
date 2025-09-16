---
tags:
  - os
  - cpu-scheduling
  - algorithms
  - process-management
created: 2025-01-08
updated: 2025-01-08
---

# CPU 스케줄링

> [!info] CPU 스케줄링이란?
> CPU 스케줄링은 운영체제가 여러 프로세스 중에서 어떤 프로세스에게 CPU를 할당할지 결정하는 메커니즘입니다. 효율적인 스케줄링은 시스템의 처리량을 높이고 응답 시간을 단축시킵니다.

## 📑 목차

- [[#🎯 스케줄링 기본 개념]]
- [[#📊 선점형 vs 비선점형]]
- [[#⏰ FCFS (First Come First Served)]]
- [[#🎯 SJF (Shortest Job First)]]
- [[#🔄 Round Robin]]
- [[#📈 Priority Scheduling]]
- [[#💡 TypeScript 구현]]
- [[#🔍 실제 사례]]

---

## 🎯 스케줄링 기본 개념

### 스케줄링 성능 지표

```typescript
interface SchedulingMetrics {
  turnaroundTime: number;    // 작업 완료까지의 총 시간
  waitingTime: number;       // 대기열에서 기다린 시간
  responseTime: number;      // 첫 응답까지의 시간
  cpuUtilization: number;    // CPU 이용률
  throughput: number;        // 단위 시간당 완료된 작업 수
}

class ProcessInfo {
  pid: number;
  arrivalTime: number;
  burstTime: number;
  priority: number;
  remainingTime: number;
  waitingTime: number = 0;
  turnaroundTime: number = 0;
  responseTime: number = -1;
  completionTime: number = 0;

  constructor(pid: number, arrivalTime: number, burstTime: number, priority: number = 0) {
    this.pid = pid;
    this.arrivalTime = arrivalTime;
    this.burstTime = burstTime;
    this.priority = priority;
    this.remainingTime = burstTime;
  }
}

abstract class CPUScheduler {
  protected processes: ProcessInfo[] = [];
  protected currentTime: number = 0;
  protected ganttChart: { pid: number; start: number; end: number; }[] = [];

  addProcess(process: ProcessInfo): void {
    this.processes.push(process);
  }

  abstract schedule(): void;

  calculateMetrics(): SchedulingMetrics {
    let totalTurnaround = 0;
    let totalWaiting = 0;
    let totalResponse = 0;
    let firstResponse = Infinity;

    for (const process of this.processes) {
      totalTurnaround += process.turnaroundTime;
      totalWaiting += process.waitingTime;
      if (process.responseTime >= 0) {
        totalResponse += process.responseTime;
        firstResponse = Math.min(firstResponse, process.responseTime);
      }
    }

    const n = this.processes.length;
    const totalTime = Math.max(...this.processes.map(p => p.completionTime));
    
    return {
      turnaroundTime: totalTurnaround / n,
      waitingTime: totalWaiting / n,
      responseTime: totalResponse / n,
      cpuUtilization: (this.processes.reduce((sum, p) => sum + p.burstTime, 0) / totalTime) * 100,
      throughput: n / totalTime
    };
  }

  printGanttChart(): void {
    console.log('\n=== Gantt Chart ===');
    let chart = '|';
    let timeline = '0';
    
    for (const entry of this.ganttChart) {
      chart += ` P${entry.pid} |`;
      timeline += `    ${entry.end}`;
    }
    
    console.log(chart);
    console.log(timeline);
  }

  printMetrics(): void {
    const metrics = this.calculateMetrics();
    console.log('\n=== Performance Metrics ===');
    console.log(`Average Turnaround Time: ${metrics.turnaroundTime.toFixed(2)}`);
    console.log(`Average Waiting Time: ${metrics.waitingTime.toFixed(2)}`);
    console.log(`Average Response Time: ${metrics.responseTime.toFixed(2)}`);
    console.log(`CPU Utilization: ${metrics.cpuUtilization.toFixed(2)}%`);
    console.log(`Throughput: ${metrics.throughput.toFixed(2)} processes/unit time`);
  }

  protected updateProcessMetrics(process: ProcessInfo): void {
    process.turnaroundTime = process.completionTime - process.arrivalTime;
    process.waitingTime = process.turnaroundTime - process.burstTime;
  }
}
```

---

## 📊 선점형 vs 비선점형

### 선점형 vs 비선점형 특성

```typescript
enum SchedulingType {
  PREEMPTIVE = 'PREEMPTIVE',
  NON_PREEMPTIVE = 'NON_PREEMPTIVE'
}

class SchedulingComparison {
  static compareTypes(): void {
    console.log('\n=== Preemptive vs Non-Preemptive ===');
    
    const characteristics = {
      nonPreemptive: {
        description: '프로세스가 CPU를 자발적으로 반납할 때까지 계속 실행',
        advantages: [
          '구현이 간단',
          '컨텍스트 스위칭 오버헤드 적음',
          '일괄 처리 시스템에 적합'
        ],
        disadvantages: [
          '응답 시간 예측 불가',
          '대화형 시스템에 부적합',
          '기아 현상 가능'
        ],
        examples: ['FCFS', 'SJF (비선점)', 'Priority (비선점)']
      },
      preemptive: {
        description: '운영체제가 강제로 프로세스를 중단시킬 수 있음',
        advantages: [
          '응답 시간 개선',
          '대화형 시스템에 적합',
          '시분할 시스템 가능'
        ],
        disadvantages: [
          '구현 복잡',
          '컨텍스트 스위칭 오버헤드',
          '동기화 문제 발생 가능'
        ],
        examples: ['Round Robin', 'SRTF', 'Priority (선점)', 'Multilevel Queue']
      }
    };

    Object.entries(characteristics).forEach(([type, char]) => {
      console.log(`\n${type.toUpperCase()}:`);
      console.log(`  ${char.description}`);
      console.log(`  장점: ${char.advantages.join(', ')}`);
      console.log(`  단점: ${char.disadvantages.join(', ')}`);
      console.log(`  예시: ${char.examples.join(', ')}`);
    });
  }
}
```

---

## ⏰ FCFS (First Come First Served)

### FCFS 구현

```typescript
class FCFSScheduler extends CPUScheduler {
  private schedulingType: SchedulingType;

  constructor(type: SchedulingType = SchedulingType.NON_PREEMPTIVE) {
    super();
    this.schedulingType = type;
  }

  schedule(): void {
    // 도착 시간 순으로 정렬
    this.processes.sort((a, b) => a.arrivalTime - b.arrivalTime);
    
    this.currentTime = 0;
    this.ganttChart = [];

    console.log('\n=== FCFS Scheduling ===');
    console.log('Process execution order based on arrival time');

    for (const process of this.processes) {
      // 프로세스가 도착할 때까지 대기
      if (this.currentTime < process.arrivalTime) {
        this.currentTime = process.arrivalTime;
      }

      // 첫 번째 응답 시간 기록
      if (process.responseTime === -1) {
        process.responseTime = this.currentTime - process.arrivalTime;
      }

      const startTime = this.currentTime;
      this.currentTime += process.burstTime;
      process.completionTime = this.currentTime;

      // Gantt 차트에 추가
      this.ganttChart.push({
        pid: process.pid,
        start: startTime,
        end: this.currentTime
      });

      this.updateProcessMetrics(process);
      
      console.log(`P${process.pid}: ${startTime} -> ${this.currentTime}`);
    }
  }

  // FCFS의 문제점 시연
  demonstrateConvoyEffect(): void {
    console.log('\n=== Convoy Effect Demonstration ===');
    console.log('Long process followed by short processes');
    
    // 긴 프로세스 하나와 짧은 프로세스들
    const testProcesses = [
      new ProcessInfo(1, 0, 10, 0),  // 긴 프로세스
      new ProcessInfo(2, 1, 1, 0),   // 짧은 프로세스
      new ProcessInfo(3, 2, 1, 0),   // 짧은 프로세스
      new ProcessInfo(4, 3, 1, 0)    // 짧은 프로세스
    ];

    const fcfs = new FCFSScheduler();
    testProcesses.forEach(p => fcfs.addProcess(p));
    fcfs.schedule();
    fcfs.printGanttChart();
    fcfs.printMetrics();
    
    console.log('\nConvoy Effect: Short processes wait for long process');
    console.log('Average waiting time is high due to the long first process');
  }
}
```

---

## 🎯 SJF (Shortest Job First)

### SJF 구현

```typescript
class SJFScheduler extends CPUScheduler {
  private isPreemptive: boolean;

  constructor(isPreemptive: boolean = false) {
    super();
    this.isPreemptive = isPreemptive;
  }

  schedule(): void {
    if (this.isPreemptive) {
      this.schedulePreemptive();
    } else {
      this.scheduleNonPreemptive();
    }
  }

  private scheduleNonPreemptive(): void {
    console.log('\n=== SJF Non-Preemptive Scheduling ===');
    
    const readyQueue: ProcessInfo[] = [];
    const allProcesses = [...this.processes];
    this.currentTime = 0;
    this.ganttChart = [];

    // 도착 시간으로 정렬
    allProcesses.sort((a, b) => a.arrivalTime - b.arrivalTime);
    
    while (allProcesses.length > 0 || readyQueue.length > 0) {
      // 현재 시간에 도착한 프로세스들을 ready queue에 추가
      while (allProcesses.length > 0 && allProcesses[0].arrivalTime <= this.currentTime) {
        readyQueue.push(allProcesses.shift()!);
      }

      if (readyQueue.length === 0) {
        // 아직 도착하지 않은 프로세스가 있으면 시간을 앞당김
        if (allProcesses.length > 0) {
          this.currentTime = allProcesses[0].arrivalTime;
        }
        continue;
      }

      // Burst time이 가장 짧은 프로세스 선택
      readyQueue.sort((a, b) => a.burstTime - b.burstTime);
      const process = readyQueue.shift()!;

      // 첫 번째 응답 시간 기록
      if (process.responseTime === -1) {
        process.responseTime = this.currentTime - process.arrivalTime;
      }

      const startTime = this.currentTime;
      this.currentTime += process.burstTime;
      process.completionTime = this.currentTime;

      this.ganttChart.push({
        pid: process.pid,
        start: startTime,
        end: this.currentTime
      });

      this.updateProcessMetrics(process);
      console.log(`P${process.pid}: ${startTime} -> ${this.currentTime} (burst: ${process.burstTime})`);
    }
  }

  private schedulePreemptive(): void {
    console.log('\n=== SRTF (Shortest Remaining Time First) Scheduling ===');
    
    const allProcesses = [...this.processes];
    const readyQueue: ProcessInfo[] = [];
    let currentProcess: ProcessInfo | null = null;
    this.currentTime = 0;
    this.ganttChart = [];

    allProcesses.sort((a, b) => a.arrivalTime - b.arrivalTime);

    while (allProcesses.length > 0 || readyQueue.length > 0 || currentProcess) {
      // 현재 시간에 도착한 프로세스들을 ready queue에 추가
      while (allProcesses.length > 0 && allProcesses[0].arrivalTime <= this.currentTime) {
        const newProcess = allProcesses.shift()!;
        readyQueue.push(newProcess);
        
        // 선점 확인
        if (currentProcess && newProcess.remainingTime < currentProcess.remainingTime) {
          // 현재 프로세스를 선점
          readyQueue.push(currentProcess);
          currentProcess = null;
        }
      }

      // 현재 실행 중인 프로세스가 없으면 새로 선택
      if (!currentProcess) {
        if (readyQueue.length === 0) {
          if (allProcesses.length > 0) {
            this.currentTime = allProcesses[0].arrivalTime;
          }
          continue;
        }

        readyQueue.sort((a, b) => a.remainingTime - b.remainingTime);
        currentProcess = readyQueue.shift()!;
        
        if (currentProcess.responseTime === -1) {
          currentProcess.responseTime = this.currentTime - currentProcess.arrivalTime;
        }
      }

      // 1 시간 단위로 실행
      const startTime = this.currentTime;
      this.currentTime++;
      currentProcess.remainingTime--;

      // 프로세스 완료 확인
      if (currentProcess.remainingTime === 0) {
        currentProcess.completionTime = this.currentTime;
        this.updateProcessMetrics(currentProcess);
        
        this.ganttChart.push({
          pid: currentProcess.pid,
          start: startTime,
          end: this.currentTime
        });
        
        console.log(`P${currentProcess.pid} completed at time ${this.currentTime}`);
        currentProcess = null;
      }
    }
  }

  // SJF의 기아 현상 시연
  demonstrateStarvation(): void {
    console.log('\n=== SJF Starvation Demonstration ===');
    
    const testProcesses = [
      new ProcessInfo(1, 0, 10, 0),  // 긴 프로세스
      new ProcessInfo(2, 1, 1, 0),   // 짧은 프로세스 (계속 도착)
      new ProcessInfo(3, 2, 1, 0),
      new ProcessInfo(4, 3, 1, 0),
      new ProcessInfo(5, 4, 1, 0)
    ];

    const sjf = new SJFScheduler(false);
    testProcesses.forEach(p => sjf.addProcess(p));
    sjf.schedule();
    sjf.printMetrics();
    
    console.log('P1 (long process) may starve if short processes keep arriving');
  }
}
```

---

## 🔄 Round Robin

### Round Robin 구현

```typescript
class RoundRobinScheduler extends CPUScheduler {
  private timeQuantum: number;

  constructor(timeQuantum: number = 4) {
    super();
    this.timeQuantum = timeQuantum;
  }

  schedule(): void {
    console.log(`\n=== Round Robin Scheduling (Time Quantum: ${this.timeQuantum}) ===`);
    
    const readyQueue: ProcessInfo[] = [];
    const allProcesses = [...this.processes];
    this.currentTime = 0;
    this.ganttChart = [];

    // 도착 시간으로 정렬
    allProcesses.sort((a, b) => a.arrivalTime - b.arrivalTime);

    // 초기에 도착한 프로세스들을 큐에 추가
    while (allProcesses.length > 0 && allProcesses[0].arrivalTime <= this.currentTime) {
      readyQueue.push(allProcesses.shift()!);
    }

    while (readyQueue.length > 0) {
      const process = readyQueue.shift()!;
      
      // 첫 번째 응답 시간 기록
      if (process.responseTime === -1) {
        process.responseTime = this.currentTime - process.arrivalTime;
      }

      const startTime = this.currentTime;
      const executionTime = Math.min(this.timeQuantum, process.remainingTime);
      
      this.currentTime += executionTime;
      process.remainingTime -= executionTime;

      // 실행 중에 도착한 프로세스들을 큐에 추가
      while (allProcesses.length > 0 && allProcesses[0].arrivalTime <= this.currentTime) {
        readyQueue.push(allProcesses.shift()!);
      }

      this.ganttChart.push({
        pid: process.pid,
        start: startTime,
        end: this.currentTime
      });

      if (process.remainingTime === 0) {
        // 프로세스 완료
        process.completionTime = this.currentTime;
        this.updateProcessMetrics(process);
        console.log(`P${process.pid} completed at time ${this.currentTime}`);
      } else {
        // 프로세스가 아직 남아있으면 큐 뒤로
        readyQueue.push(process);
        console.log(`P${process.pid} preempted at time ${this.currentTime} (remaining: ${process.remainingTime})`);
      }
    }
  }

  // Time Quantum 영향 분석
  analyzeTimeQuantumEffect(processes: ProcessInfo[]): void {
    console.log('\n=== Time Quantum Effect Analysis ===');
    
    const quantums = [1, 2, 4, 8, 16];
    
    for (const quantum of quantums) {
      // 프로세스 초기화
      const testProcesses = processes.map(p => new ProcessInfo(p.pid, p.arrivalTime, p.burstTime, p.priority));
      
      const rr = new RoundRobinScheduler(quantum);
      testProcesses.forEach(p => rr.addProcess(p));
      rr.schedule();
      
      const metrics = rr.calculateMetrics();
      console.log(`Quantum ${quantum}: Avg Turnaround = ${metrics.turnaroundTime.toFixed(2)}, ` +
                 `Context Switches = ${rr.ganttChart.length}`);
    }
    
    console.log('\nObservations:');
    console.log('- Small quantum: Better response time, more context switches');
    console.log('- Large quantum: Approaches FCFS, fewer context switches');
    console.log('- Optimal quantum: 10-100ms in practice');
  }
}
```

---

## 📈 Priority Scheduling

### Priority Scheduling 구현

```typescript
class PriorityScheduler extends CPUScheduler {
  private isPreemptive: boolean;
  private useAging: boolean;
  private agingFactor: number;

  constructor(isPreemptive: boolean = false, useAging: boolean = true, agingFactor: number = 1) {
    super();
    this.isPreemptive = isPreemptive;
    this.useAging = useAging;
    this.agingFactor = agingFactor;
  }

  schedule(): void {
    if (this.isPreemptive) {
      this.schedulePreemptive();
    } else {
      this.scheduleNonPreemptive();
    }
  }

  private scheduleNonPreemptive(): void {
    console.log('\n=== Priority Non-Preemptive Scheduling ===');
    console.log('Lower number = Higher priority');
    
    const readyQueue: ProcessInfo[] = [];
    const allProcesses = [...this.processes];
    this.currentTime = 0;
    this.ganttChart = [];

    allProcesses.sort((a, b) => a.arrivalTime - b.arrivalTime);

    while (allProcesses.length > 0 || readyQueue.length > 0) {
      // 현재 시간에 도착한 프로세스들을 ready queue에 추가
      while (allProcesses.length > 0 && allProcesses[0].arrivalTime <= this.currentTime) {
        readyQueue.push(allProcesses.shift()!);
      }

      if (readyQueue.length === 0) {
        if (allProcesses.length > 0) {
          this.currentTime = allProcesses[0].arrivalTime;
        }
        continue;
      }

      // Aging 적용
      if (this.useAging) {
        this.applyAging(readyQueue);
      }

      // 우선순위가 가장 높은 프로세스 선택 (낮은 숫자 = 높은 우선순위)
      readyQueue.sort((a, b) => {
        if (a.priority !== b.priority) {
          return a.priority - b.priority;
        }
        // 우선순위가 같으면 도착 시간 순
        return a.arrivalTime - b.arrivalTime;
      });
      
      const process = readyQueue.shift()!;

      if (process.responseTime === -1) {
        process.responseTime = this.currentTime - process.arrivalTime;
      }

      const startTime = this.currentTime;
      this.currentTime += process.burstTime;
      process.completionTime = this.currentTime;

      this.ganttChart.push({
        pid: process.pid,
        start: startTime,
        end: this.currentTime
      });

      this.updateProcessMetrics(process);
      console.log(`P${process.pid} (priority: ${process.priority}): ${startTime} -> ${this.currentTime}`);
    }
  }

  private schedulePreemptive(): void {
    console.log('\n=== Priority Preemptive Scheduling ===');
    
    const allProcesses = [...this.processes];
    const readyQueue: ProcessInfo[] = [];
    let currentProcess: ProcessInfo | null = null;
    this.currentTime = 0;
    this.ganttChart = [];

    allProcesses.sort((a, b) => a.arrivalTime - b.arrivalTime);

    while (allProcesses.length > 0 || readyQueue.length > 0 || currentProcess) {
      // 현재 시간에 도착한 프로세스들을 ready queue에 추가
      while (allProcesses.length > 0 && allProcesses[0].arrivalTime <= this.currentTime) {
        const newProcess = allProcesses.shift()!;
        readyQueue.push(newProcess);
        
        // 선점 확인
        if (currentProcess && newProcess.priority < currentProcess.priority) {
          readyQueue.push(currentProcess);
          currentProcess = null;
        }
      }

      // Aging 적용
      if (this.useAging && readyQueue.length > 0) {
        this.applyAging(readyQueue);
      }

      if (!currentProcess) {
        if (readyQueue.length === 0) {
          if (allProcesses.length > 0) {
            this.currentTime = allProcesses[0].arrivalTime;
          }
          continue;
        }

        readyQueue.sort((a, b) => a.priority - b.priority);
        currentProcess = readyQueue.shift()!;
        
        if (currentProcess.responseTime === -1) {
          currentProcess.responseTime = this.currentTime - currentProcess.arrivalTime;
        }
      }

      const startTime = this.currentTime;
      this.currentTime++;
      currentProcess.remainingTime--;

      if (currentProcess.remainingTime === 0) {
        currentProcess.completionTime = this.currentTime;
        this.updateProcessMetrics(currentProcess);
        
        this.ganttChart.push({
          pid: currentProcess.pid,
          start: startTime,
          end: this.currentTime
        });
        
        console.log(`P${currentProcess.pid} completed at time ${this.currentTime}`);
        currentProcess = null;
      }
    }
  }

  private applyAging(readyQueue: ProcessInfo[]): void {
    // 대기 시간에 비례하여 우선순위 증가 (숫자 감소)
    for (const process of readyQueue) {
      const waitingTime = this.currentTime - process.arrivalTime;
      const ageBonus = Math.floor(waitingTime / 10) * this.agingFactor;
      process.priority = Math.max(0, process.priority - ageBonus);
    }
  }

  // 우선순위 역전 문제 시연
  demonstratePriorityInversion(): void {
    console.log('\n=== Priority Inversion Problem ===');
    
    // 높은 우선순위 프로세스가 낮은 우선순위 프로세스를 기다리는 상황
    const testProcesses = [
      new ProcessInfo(1, 0, 8, 3),   // 낮은 우선순위 (높은 숫자)
      new ProcessInfo(2, 2, 2, 1),   // 높은 우선순위 (낮은 숫자)
      new ProcessInfo(3, 4, 4, 2)    // 중간 우선순위
    ];

    const ps = new PriorityScheduler(true, false);
    testProcesses.forEach(p => ps.addProcess(p));
    ps.schedule();
    ps.printGanttChart();
    
    console.log('Priority Inversion: High priority process may wait for low priority process');
    console.log('Solution: Priority Inheritance Protocol or Priority Ceiling Protocol');
  }
}
```

---

## 💡 TypeScript 구현

### 고급 스케줄링 알고리즘

```typescript
// Multilevel Queue Scheduling
class MultilevelQueueScheduler extends CPUScheduler {
  private queues: Map<string, ProcessInfo[]> = new Map();
  private queueSchedulers: Map<string, CPUScheduler> = new Map();
  private queuePriorities: string[] = [];

  constructor() {
    super();
    this.initializeQueues();
  }

  private initializeQueues(): void {
    // 시스템 프로세스 큐 (높은 우선순위)
    this.queues.set('system', []);
    this.queueSchedulers.set('system', new FCFSScheduler());

    // 대화형 프로세스 큐
    this.queues.set('interactive', []);
    this.queueSchedulers.set('interactive', new RoundRobinScheduler(4));

    // 배치 프로세스 큐 (낮은 우선순위)
    this.queues.set('batch', []);
    this.queueSchedulers.set('batch', new FCFSScheduler());

    this.queuePriorities = ['system', 'interactive', 'batch'];
  }

  addProcessToQueue(process: ProcessInfo, queueType: string): void {
    const queue = this.queues.get(queueType);
    if (queue) {
      queue.push(process);
      this.processes.push(process);
    }
  }

  schedule(): void {
    console.log('\n=== Multilevel Queue Scheduling ===');
    
    // 각 큐를 우선순위 순으로 처리
    for (const queueType of this.queuePriorities) {
      const queue = this.queues.get(queueType)!;
      if (queue.length > 0) {
        console.log(`\nProcessing ${queueType} queue:`);
        
        const scheduler = this.queueSchedulers.get(queueType)!;
        queue.forEach(p => scheduler.addProcess(p));
        scheduler.schedule();
        
        // 결과를 메인 스케줄러에 반영
        this.ganttChart.push(...scheduler.ganttChart);
      }
    }
  }
}

// Multilevel Feedback Queue
class MultilevelFeedbackQueueScheduler extends CPUScheduler {
  private queues: ProcessInfo[][] = [];
  private timeQuantums: number[] = [];
  private currentTime: number = 0;

  constructor() {
    super();
    this.initializeQueues();
  }

  private initializeQueues(): void {
    // 3개의 큐 생성 (우선순위 높음 -> 낮음)
    this.queues = [[], [], []];
    this.timeQuantums = [2, 4, 8]; // 큐별 time quantum
  }

  schedule(): void {
    console.log('\n=== Multilevel Feedback Queue Scheduling ===');
    console.log('Queue 0: RR with quantum 2');
    console.log('Queue 1: RR with quantum 4');  
    console.log('Queue 2: RR with quantum 8');
    
    // 모든 프로세스를 최고 우선순위 큐에 추가
    this.processes.forEach(p => this.queues[0].push(p));
    
    this.currentTime = 0;
    this.ganttChart = [];

    while (this.hasProcesses()) {
      let processExecuted = false;
      
      // 우선순위 순으로 큐 확인
      for (let queueIndex = 0; queueIndex < this.queues.length; queueIndex++) {
        const queue = this.queues[queueIndex];
        
        if (queue.length > 0) {
          const process = queue.shift()!;
          const quantum = this.timeQuantums[queueIndex];
          
          if (process.responseTime === -1) {
            process.responseTime = this.currentTime - process.arrivalTime;
          }

          const startTime = this.currentTime;
          const executionTime = Math.min(quantum, process.remainingTime);
          
          this.currentTime += executionTime;
          process.remainingTime -= executionTime;

          this.ganttChart.push({
            pid: process.pid,
            start: startTime,
            end: this.currentTime
          });

          if (process.remainingTime === 0) {
            // 프로세스 완료
            process.completionTime = this.currentTime;
            this.updateProcessMetrics(process);
            console.log(`P${process.pid} completed in queue ${queueIndex} at time ${this.currentTime}`);
          } else {
            // 다음 큐로 이동 (demotion)
            const nextQueue = Math.min(queueIndex + 1, this.queues.length - 1);
            this.queues[nextQueue].push(process);
            console.log(`P${process.pid} moved from queue ${queueIndex} to queue ${nextQueue}`);
          }
          
          processExecuted = true;
          break;
        }
      }

      if (!processExecuted) break;
    }
  }

  private hasProcesses(): boolean {
    return this.queues.some(queue => queue.length > 0);
  }
}

// Fair Share Scheduling
class FairShareScheduler extends CPUScheduler {
  private userGroups: Map<string, ProcessInfo[]> = new Map();
  private groupShares: Map<string, number> = new Map();

  addUserProcess(process: ProcessInfo, userId: string, share: number): void {
    if (!this.userGroups.has(userId)) {
      this.userGroups.set(userId, []);
      this.groupShares.set(userId, share);
    }
    
    this.userGroups.get(userId)!.push(process);
    this.processes.push(process);
  }

  schedule(): void {
    console.log('\n=== Fair Share Scheduling ===');
    
    const totalShare = Array.from(this.groupShares.values()).reduce((sum, share) => sum + share, 0);
    
    this.currentTime = 0;
    this.ganttChart = [];

    while (this.hasProcesses()) {
      // 각 그룹의 점유율에 따라 CPU 시간 할당
      for (const [userId, processes] of this.userGroups) {
        if (processes.length === 0) continue;
        
        const groupShare = this.groupShares.get(userId)!;
        const timeSlice = Math.floor((groupShare / totalShare) * 10); // 10 단위 시간
        
        const process = processes.shift()!;
        
        if (process.responseTime === -1) {
          process.responseTime = this.currentTime - process.arrivalTime;
        }

        const startTime = this.currentTime;
        const executionTime = Math.min(timeSlice, process.remainingTime);
        
        this.currentTime += executionTime;
        process.remainingTime -= executionTime;

        this.ganttChart.push({
          pid: process.pid,
          start: startTime,
          end: this.currentTime
        });

        if (process.remainingTime === 0) {
          process.completionTime = this.currentTime;
          this.updateProcessMetrics(process);
          console.log(`P${process.pid} (User: ${userId}) completed at ${this.currentTime}`);
        } else {
          processes.push(process); // 큐 뒤로
        }
      }
    }
  }

  private hasProcesses(): boolean {
    return Array.from(this.userGroups.values()).some(processes => processes.length > 0);
  }
}
```

---

## 🔍 실제 사례

### 실제 운영체제의 스케줄링

```typescript
// Linux CFS (Completely Fair Scheduler)
class LinuxCFS {
  static getDescription(): string {
    return `
Linux CFS (Completely Fair Scheduler):
- Red-Black Tree 기반 구현
- Virtual Runtime (vruntime) 사용
- Nice 값으로 우선순위 조정
- O(log n) 복잡도
- 실시간 특성 고려한 설계
    `;
  }

  static simulateVirtualRuntime(): void {
    console.log('\n=== Linux CFS Virtual Runtime Simulation ===');
    
    interface CFSProcess {
      pid: number;
      vruntime: number;
      weight: number;
      nice: number;
    }

    const processes: CFSProcess[] = [
      { pid: 1, vruntime: 100, weight: 1024, nice: 0 },   // 기본 우선순위
      { pid: 2, vruntime: 150, weight: 820, nice: 5 },    // 낮은 우선순위
      { pid: 3, vruntime: 80, weight: 1277, nice: -5 }    // 높은 우선순위
    ];

    console.log('Initial virtual runtimes:');
    processes.forEach(p => 
      console.log(`P${p.pid}: vruntime=${p.vruntime}, nice=${p.nice}, weight=${p.weight}`)
    );

    // 가장 작은 vruntime을 가진 프로세스 선택
    processes.sort((a, b) => a.vruntime - b.vruntime);
    const selected = processes[0];
    
    console.log(`\nSelected process: P${selected.pid} (lowest vruntime)`);
    
    // vruntime 업데이트 공식: vruntime += runtime * (1024 / weight)
    const runtime = 10; // 10ms 실행
    selected.vruntime += runtime * (1024 / selected.weight);
    
    console.log(`After ${runtime}ms execution: vruntime=${selected.vruntime}`);
  }
}

// Windows Scheduler
class WindowsScheduler {
  static getDescription(): string {
    return `
Windows Process Scheduler:
- 32 Priority Levels (0-31)
- Multilevel Feedback Queue
- Priority Boost for I/O completion
- Quantum varies by system type
- Real-time priority class (16-31)
    `;
  }

  static simulatePriorityBoost(): void {
    console.log('\n=== Windows Priority Boost Simulation ===');
    
    interface WindowsProcess {
      pid: number;
      basePriority: number;
      currentPriority: number;
      quantum: number;
    }

    const process: WindowsProcess = {
      pid: 123,
      basePriority: 8,
      currentPriority: 8,
      quantum: 6
    };

    console.log(`Process P${process.pid}:`);
    console.log(`Base Priority: ${process.basePriority}`);
    console.log(`Current Priority: ${process.currentPriority}`);
    
    // I/O 완료 시 우선순위 부스트
    const ioBoost = 2;
    process.currentPriority = Math.min(15, process.currentPriority + ioBoost);
    
    console.log(`After I/O completion boost: ${process.currentPriority}`);
    
    // 시간이 지나면서 점진적으로 base priority로 회복
    for (let tick = 1; tick <= 3; tick++) {
      if (process.currentPriority > process.basePriority) {
        process.currentPriority--;
        console.log(`Tick ${tick}: Priority decayed to ${process.currentPriority}`);
      }
    }
  }
}

// Real-time Scheduling
class RealTimeScheduler extends CPUScheduler {
  private hardRealTime: ProcessInfo[] = [];
  private softRealTime: ProcessInfo[] = [];

  addRealTimeProcess(process: ProcessInfo, isHard: boolean = false): void {
    if (isHard) {
      this.hardRealTime.push(process);
    } else {
      this.softRealTime.push(process);
    }
    this.processes.push(process);
  }

  schedule(): void {
    console.log('\n=== Real-Time Scheduling (EDF - Earliest Deadline First) ===');
    
    // 데드라인 순으로 정렬
    const allRTProcesses = [...this.hardRealTime, ...this.softRealTime];
    allRTProcesses.sort((a, b) => (a.arrivalTime + a.burstTime) - (b.arrivalTime + b.burstTime));
    
    this.currentTime = 0;
    this.ganttChart = [];

    for (const process of allRTProcesses) {
      const deadline = process.arrivalTime + process.burstTime * 2; // 예시 데드라인
      
      if (this.currentTime < process.arrivalTime) {
        this.currentTime = process.arrivalTime;
      }

      if (process.responseTime === -1) {
        process.responseTime = this.currentTime - process.arrivalTime;
      }

      const startTime = this.currentTime;
      this.currentTime += process.burstTime;
      process.completionTime = this.currentTime;

      this.ganttChart.push({
        pid: process.pid,
        start: startTime,
        end: this.currentTime
      });

      this.updateProcessMetrics(process);

      const missedDeadline = this.currentTime > deadline;
      console.log(
        `P${process.pid}: ${startTime} -> ${this.currentTime} ` +
        `(deadline: ${deadline}${missedDeadline ? ' - MISSED!' : ' - OK'})`
      );
    }
  }
}
```

> [!tip] 실무 팁
> CPU 스케줄링 선택 시 시스템 특성을 고려하세요. 대화형 시스템에는 Round Robin이나 Multilevel Feedback Queue, 일괄 처리에는 FCFS나 SJF, 실시간 시스템에는 EDF나 Rate Monotonic이 적합합니다. 또한 공정성과 성능의 균형을 맞추는 것이 중요합니다.

## 📚 참고 자료
- Operating System Concepts (Silberschatz)
- Modern Operating Systems (Tanenbaum)
- Linux Kernel Development (Love)
- Windows Internals (Russinovich)