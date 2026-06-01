# 📐 Low-Level Design: Observer Pattern Blueprint

## 1. Intent & Context
Establish loose architectural coupling across internal functional boundaries. Allows core mutations (e.g., an order changing state to `PAID`) to broadcast reactions to auxiliary secondary operations (e.g., stock adjustment, WhatsApp alerts) without direct code dependency chains.

## 2. Core Abstract Interface Implementation
```python
from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    async def update(self, event_type: str, data: dict) -> None:
        pass

class Subject(ABC):
    def __init__(self):
        self._observers: List[Observer] = []

    def attach(self, observer: Observer) -> None:
        if observer not in self._observers:
            self._observers.append(observer)

    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)

    @abstractmethod
    async def notify(self, event_type: str, data: dict) -> None:
        pass
```

## 3. Concrete Implementations
```python
class OrderProcessor(Subject):
    async def notify(self, event_type: str, data: dict) -> None:
        for observer in self._observers:
            await observer.update(event_type, data)

    async def mark_as_paid(self, order_id: str, payload: dict) -> None:
        # Core domain persistence execution completes here...
        print(f"Order {order_id} mutated to PAID state.")
        await self.notify("ORDER_PAID", {"order_id": order_id, "meta": payload})

class InventoryObserver(Observer):
    async def update(self, event_type: str, data: dict) -> None:
        if event_type == "ORDER_PAID":
            print(f"Decrementing regional stock allocations for order: {data['order_id']}")

class WhatsAppNotificationObserver(Observer):
    async def update(self, event_type: str, data: dict) -> None:
        if event_type == "ORDER_PAID":
            print(f"Enqueuing WhatsApp delivery updates via WATI outbox rules.")
```

## 4. Engineering Checklists
*   [ ] **Asynchronous Task Demarcation**: Never allow an observer's runtime exception to crash the primary thread. Wrap notifications inside non-blocking event loops or dispatch to asynchronous worker threads.
*   [ ] **Memory Leak Avoidance**: Ensure short-lived entities detach cleanly from subjects to prevent garbage collection failures.
