# 📐 Low-Level Design: Strategy Pattern Blueprint

## 1. Intent & Context
Decouple execution workflows from core domain logic. Essential for building systems where components have multiple implementation options (e.g., dynamic payment routing, multi-channel notifications) without inducing code smell via nested conditional evaluations (`if/else`).

## 2. Core Abstract Interface Implementation
```python
from abc import ABC, abstractmethod
from typing import Dict, Any

class PaymentStrategy(ABC):
    """
    Immutable Strategy Interface. All concrete engine implementations 
    must comply with this execution contract.
    """
    @abstractmethod
    async def process_transaction(self, amount: float, order_id: str) -> Dict[str, Any]:
        pass
```

## 3. Concrete Implementations
```python
class RazorpayStrategy(PaymentStrategy):
    def __init__(self, key_id: str, secret: str):
        self.key_id = key_id
        self.secret = secret

    async def process_transaction(self, amount: float, order_id: str) -> Dict[str, Any]:
        # Invoke Razorpay SDK / API Network request
        return {"status": "success", "transaction_id": f"rzp_{order_id}"}

class CashOnDeliveryStrategy(PaymentStrategy):
    async def process_transaction(self, amount: float, order_id: str) -> Dict[str, Any]:
        return {"status": "pending_delivery", "transaction_id": f"cod_{order_id}"}
```

## 4. Context Execution Engine
```python
class PaymentContext:
    def __init__(self, strategy: PaymentStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: PaymentStrategy) -> None:
        self._strategy = strategy

    async def execute_payment(self, amount: float, order_id: str) -> Dict[str, Any]:
        # Domain validation logic executes here before invoking strategy runtime
        if amount <= 0:
            raise ValueError("Invalid transaction valuation limits.")
        return await self._strategy.process_transaction(amount, order_id)
```

## 5. Engineering Checklists
*   [ ] **Strict Interface Contracts**: Never pass freeform parameters or variable dictionaries into internal strategy methods. Use rigid typing or Pydantic validation structures.
*   [ ] **Runtime Swapping Safeguards**: Ensure switching strategy models mid-execution thread is atomic and doesn't leak memory pointers or state metrics.
