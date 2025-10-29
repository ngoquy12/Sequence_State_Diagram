sequenceDiagram
autonumber
actor Customer as Khách hàng
participant UI as Web/App
participant OrderSvc as Order Service
participant Inv as Inventory Service
participant PG as Payment Gateway
participant Mail as Email Service
participant Ship as Delivery Service

Customer->>UI: Đặt hàng (tạo đơn)
UI->>OrderSvc: createOrder(payload)
OrderSvc->>Inv: reserveStock(items)
alt Reserve OK
    Inv-->>OrderSvc: reserved=true
    OrderSvc-->>UI: Trả về draft order (state=NEW)
    UI-->>Customer: Hiển thị chờ thanh toán
    Customer->>PG: Thanh toán (pay)
    PG-->>OrderSvc: paymentResult(success/fail)
    alt success
        OrderSvc->>OrderSvc: updateState(PAID -> SHIPPING)
        OrderSvc->>Ship: createShipment(orderId)
        Ship-->>OrderSvc: shipmentCreated
        OrderSvc->>Mail: sendOrderConfirmedEmail()
        Mail-->>OrderSvc: mailed
        Ship-->>OrderSvc: delivered
        OrderSvc->>OrderSvc: updateState(SHIPPING -> COMPLETED)
        OrderSvc-->>UI: notifyCompleted
        UI-->>Customer: Hiển thị Hoàn thành
    else fail
        OrderSvc->>Inv: releaseStock(items)
        OrderSvc->>OrderSvc: updateState(NEW/WAITING_PAYMENT -> CANCELED)
        OrderSvc->>Mail: sendOrderCanceledEmail()
        Mail-->>OrderSvc: mailed
        OrderSvc-->>UI: notifyCanceled
        UI-->>Customer: Thông báo thanh toán thất bại
    end
else Reserve FAIL
    Inv-->>OrderSvc: reserved=false
    OrderSvc->>OrderSvc: updateState(NEW -> CANCELED)
    OrderSvc-->>UI: Notify out-of-stock
    UI-->>Customer: Thông báo hết hàng
end
