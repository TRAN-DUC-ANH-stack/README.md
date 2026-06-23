## Class Diagram: `Payment`

| Layer | Member |
|---|---|
| **Class** | `Payment` |

### Attributes (private)
| Visibility | Name | Type |
|---|---|---|
| `–` | `paymentId` | `String` |
| `–` | `invoiceId` | `String` |
| `–` | `amount` | `Double` |
| `–` | `paymentDate` | `Date` |
| `–` | `method` | `String` |

### Constructors (public)
| Signature |
|---|
| `+ Payment()` |
| `+ Payment(paymentId: String, invoiceId: String, amount: Double, paymentDate: Date, method: String)` |

### Getters & Setters (public)
| Signature | Return |
|---|---|
| `+ getPaymentId()` | `String` |
| `+ setPaymentId(paymentId: String)` | `void` |
| `+ getInvoiceId()` | `String` |
| `+ setInvoiceId(invoiceId: String)` | `void` |
| `+ getAmount()` | `Double` |
| `+ setAmount(amount: Double)` | `void` |
| `+ getPaymentDate()` | `Date` |
| `+ getDate()` | `Date` |
| `+ setPaymentDate(paymentDate: Date)` | `void` |
| `+ getMethod()` | `String` |
| `+ setMethod(method: String)` | `void` |

### Core Methods (public)
| Signature | Return | Description |
|---|---|---|
| `+ processPayment()` | `boolean` | Returns `true` if `amount > 0`, prints success message |
| `+ recordTransaction()` | `void` | Prints full transaction info to console |
| `+ getPaymentStatus()` | `String` | Returns `"Đã thanh toán"` or `"Chưa thanh toán"` |
| `+ cancelPayment()` | `void` | Sets `amount = 0.0`, prints cancellation message |
| `+ toString()` | `String` | Returns formatted string representation |

### UML Notation Key
- `–` = `private`
- `+` = `public`

## Class Diagram: `Invoice`

### Attributes (private)
| Visibility | Name | Type |
|---|---|---|
| `–` | `invoiceId` | `String` |
| `–` | `customerId` | `String` |
| `–` | `totalAmount` | `Double` |
| `–` | `issueDate` | `Date` |
| `–` | `status` | `String` |

### Constructor (public)
| Signature |
|---|
| `+ Invoice(invoiceId: String, customerId: String, totalAmount: Double, issueDate: Date, status: String)` |

### Getters & Setters (public)
| Signature | Return |
|---|---|
| `+ getInvoiceId()` | `String` |
| `+ setInvoiceId(invoiceId: String)` | `void` |
| `+ getCustomerId()` | `String` |
| `+ setCustomerId(customerId: String)` | `void` |
| `+ getTotalAmount()` | `Double` |
| `+ setTotalAmount(totalAmount: double)` | `void` |
| `+ getIssueDate()` | `Date` |
| `+ setIssueDate(issueDate: Date)` | `void` |
| `+ getStatus()` | `String` |
| `+ setStatus(status: String)` | `void` |

### Core Methods (public)
| Signature | Return | Description |
|---|---|---|
| `+ createInvoice()` | `void` | Khởi tạo hóa đơn, gán ngày hiện tại và trạng thái "chờ thanh toán" |
| `+ printInvoice()` | `void` | In thông tin hóa đơn ra màn hình console |
| `+ updateStatus(newStatus: String)` | `void` | Cập nhật trạng thái hóa đơn |
| `+ getInvoiceById(id: String)` | `Invoice` | Tìm hóa đơn theo ID, trả về `this` hoặc `null` |
| `+ toString()` | `String` | Trả về chuỗi biểu diễn đối tượng |

### UML Notation Key
- `–` = `private`
- `+` = `public`
