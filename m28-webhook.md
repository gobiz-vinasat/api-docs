# Fulfilment webhook events

- [Webhook Request](#webhook-request) 
- Events
    - [ORDER.CHANGE_STATUS](#orderchange_status) 
    - [SKU.CHANGE_STOCK](#skuchange_stock) 
- [Data Definition](#data-definition) 

## Webhook Request

### Request

- Method: `POST`
- Headers:
    - `X-Event-Delivery` (int): Delivery identity
    - `X-Event-Signature` (string): Signature dùng để xác thực thông tin event
- Body:
    - `event` (string): Event name
    - `payload` (object): Event payload

### Validating event signature

Cách tạo event signature:

`signature = sha256(request_body + webhook_secret)`

Example:

```php
function validateEventSignature() {
    $webhookSecret = 'XXX';
    $sign = request()->header('X-Event-Signature');
    $eventSignature = hash('sha256', request()->getContent() . $webhookSecret);
    
    return $eventSignature === $sign;
}
```

## ORDER.CHANGE_STATUS

Event được gửi mỗi khi đơn thay đổi trạng thái

### Webhook payload object

| Key      | Type | Description |
| --- | --- | --- |
| order | object | Thông tin đơn |
| merchant_code | string | Mã người bán |
| from_status | string | Trạng thái trước thay đổi |
| to_status | string | Trạng thái sau thay đổi |

### Webhook payload example

```json
{
  "event": "ORDER.CHANGE_STATUS",
  "payload": {
      "order": {
        "id": 5342,
        "tenant_id": 1,
        "merchant_id": 116,
        "store_id": null,
        "marketplace_code": null,
        "marketplace_store_id": null,
        "code": "dh07060099",
        "status": "WAITING_CONFIRM",
        "order_amount": 198,
        "discount_amount": 0,
        "shipping_amount": null,
        "total_amount": 198,
        "paid_amount": 0,
        "debit_amount": 198,
        "receiver_name": "sam",
        "receiver_phone": "02939823882",
        "receiver_address": "so 1",
        "receiver_note": null,
        "intended_delivery_at": null,
        "payment_time": null,
        "payment_type": "COD",
        "payment_method": null,
        "description": "",
        "customer_id": null,
        "customer_address_id": null,
        "sale_id": 0,
        "created_at": "2022-06-07T09:55:56.000000Z",
        "updated_at": "2022-06-07T09:55:58.000000Z",
        "receiver_country_id": 11935,
        "receiver_province_id": 60,
        "receiver_district_id": 718,
        "receiver_ward_id": 10112,
        "extra_services": [],
        "creator_id": 17,
        "cancel_note": null,
        "cancel_reason": "OTHER",
        "freight_bill": null,
        "campaign": null,
        "created_at_origin": null,
        "currency_id": 4,
        "shipping_partner_id": 5,
        "cod": 198,
        "finance_status": "UNPAID",
        "cod_fee_amount": null,
        "other_fee": null,
        "service_amount": null,
        "amount_paid_to_seller": 0,
        "finance_service_status": "UNPAID",
        "service_import_return_goods_amount": null,
        "finance_service_import_return_goods_status": "UNPAID",
        "cost_price": null,
        "dropship": 0,
        "warehouse_id": null,
        "inspected": true,
        "priority": 0
      },
      "merchant_code": "XYZ",
      "from_status": "WAITING_INSPECTION",
      "to_status": "WAITING_CONFIRM"
  }
}
```


## SKU.CHANGE_STOCK

Event được gửi khi sản phẩm thay đổi tồn kho

### Webhook payload object

| Key      | Type | Description |
| --- | --- | --- |
| product_code | string | Mã sản phẩm |
| merchant_code | string | Mã người bán |
| stocks | array | Danh sách thông tin tồn kho |
| stocks[*].stock | object | Thông tin tồn kho |
| stocks[*].warehouse_code | string | Mã kho |

### Webhook payload example

```json
{
  "event": "SKU.CHANGE_STOCK",
  "payload": {
    "product_code": "ABC",
    "merchant_code": "XYZ",
    "stocks": [
      {
        "stock": {
          "id": 543,
          "tenant_id": 1,
          "product_id": 29,
          "sku_id": 189,
          "warehouse_id": 25,
          "warehouse_area_id": 61,
          "quantity": 1,
          "real_quantity": 1,
          "total_storage_fee": null,
          "created_at": "2022-05-13T08:40:37.000000Z",
          "updated_at": "2022-06-06T10:35:13.000000Z",
          "real_stock": null
        },
        "warehouse_code": "XYZ"
      },
      {
        "stock": {
          "id": 544,
          "tenant_id": 1,
          "product_id": 29,
          "sku_id": 189,
          "warehouse_id": 25,
          "warehouse_area_id": 63,
          "quantity": 0,
          "real_quantity": 1,
          "total_storage_fee": null,
          "created_at": "2022-05-13T08:40:44.000000Z",
          "updated_at": "2022-06-07T10:50:23.000000Z",
          "real_stock": null
        },
        "warehouse_code": "XYZ"
      }
    ]
  }
}
```

## Data Definition

### OrderStatus

| Code | Label |
| --- | --- | 
| WAITING_INSPECTION | Chờ chọn kho |
| WAITING_CONFIRM | Chờ xác nhận |
| WAITING_PROCESSING | Chờ xử lý |
| WAITING_PICKING | Chờ nhặt hàng |
| WAITING_PACKING | Chờ đóng gói |
| WAITING_DELIVERY | Chờ giao |
| DELIVERING | Đang giao hàng |
| PART_DELIVERED | Đã giao 1 phần hàng |
| DELIVERED | Đã giao hàng |
| FINISH | Hoàn thành |
| CANCELED | Đã hủy |
| RETURN | Sẽ trả hàng hoặc đang trả hàng |
| RETURN_COMPLETED | Đã trả hàng |
| FAILED_DELIVERY | Giao hàng lỗi |