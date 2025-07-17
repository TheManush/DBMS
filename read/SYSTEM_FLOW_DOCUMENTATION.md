# TaxMate System Flow Documentation

## Table of Contents
1. [Client Request Flow](#client-request-flow)
2. [CA Approval/Rejection Process](#ca-approvalrejection-process)
3. [Chat System Architecture](#chat-system-architecture)
4. [Notification System](#notification-system)
5. [Database Schema](#database-schema)

---

## Client Request Flow

### 1. Request Initiation
**File: `tax_audit_page.dart`**
- Client browses available Chartered Accountants
- Client selects a CA from the list
- Navigation to `CA_profile_from_client.dart` page

### 2. Request Submission
**File: `CA_profile_from_client.dart`**
```dart
// Client sends request to CA
await apiService.sendRequestToCA(caId, clientId);
```

**Backend Process:**
- **Endpoint**: `POST /send-request-to-ca`
- **File**: `main.py` or similar route handler
- Creates new record in `ca_requests` table
- Initial status: `'pending'`
- Associates client and CA via their IDs

**Database Table: `ca_requests`**
```sql
CREATE TABLE ca_requests (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    ca_id INTEGER REFERENCES chartered_accountants(id),
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. Request Visibility
**Client Side:**
- Client can view sent requests in their dashboard
- Status tracking: `pending`, `approved`, `rejected`

**CA Side:**
- Requests appear in CA dashboard (`ca_dashboard.dart`)
- Fetched via `apiService.fetchCARequests(caId)`

---

## CA Approval/Rejection Process

### 1. Request Display
**File: `ca_dashboard.dart`**
```dart
Widget _buildRequestList() {
  return FutureBuilder<List<Map<String, dynamic>>>(
    future: _pendingRequests,
    builder: (context, snapshot) {
      // Displays cards with client info and action buttons
    }
  );
}
```

### 2. Action Buttons
**Approval Flow:**
```dart
ElevatedButton.icon(
  onPressed: () async {
    await widget.apiService.updateRequestStatus(req['id'], 'approved');
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Request accepted')),
    );
    _fetchRequests(); // Refresh the list
  },
  icon: const Icon(Icons.check, size: 16),
  label: const Text('Accept'),
)
```

**Rejection Flow:**
```dart
ElevatedButton.icon(
  onPressed: () async {
    await widget.apiService.updateRequestStatus(req['id'], 'rejected');
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Request rejected')),
    );
    _fetchRequests(); // Refresh the list
  },
  icon: const Icon(Icons.close, size: 16),
  label: const Text('Reject'),
)
```

### 3. Backend Processing
**File: `api_service.dart`**
```dart
Future<void> updateRequestStatus(int requestId, String status) async {
  final response = await _client.put(
    Uri.parse('$baseUrl/update-request-status/$requestId'),
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode({'status': status}),
  );
}
```

**Backend Endpoint:**
- **Route**: `PUT /update-request-status/{request_id}`
- **Action**: Updates `status` field in `ca_requests` table
- **Possible Values**: `'pending'`, `'approved'`, `'rejected'`

### 4. Status Reflection
**Database Update:**
```sql
UPDATE ca_requests 
SET status = 'approved', updated_at = CURRENT_TIMESTAMP 
WHERE id = request_id;
```

**Client View Update:**
- Status badge changes color and text
- Approved: Green badge with "APPROVED"
- Rejected: Red badge with "REJECTED"
- Pending: Orange badge with "PENDING"

---

## Chat System Architecture

### 1. Chat Page Structure
**File: `chat_page.dart`**

**Key Components:**
- Message display list
- Input field for new messages
- Real-time message updates
- User identification system

### 2. Message Flow
**Message Sending:**
```dart
// User types message and sends
void _sendMessage(String messageText) async {
  await apiService.sendMessage(
    senderId: currentUserId,
    receiverId: otherUserId,
    message: messageText,
    senderType: userType, // 'client', 'ca', 'blo', etc.
  );
  _loadMessages(); // Refresh chat
}
```

**Message Storage:**
```sql
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    sender_id INTEGER,
    receiver_id INTEGER,
    sender_type VARCHAR(20), -- 'client', 'ca', 'blo', 'fp'
    receiver_type VARCHAR(20),
    message_text TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_read BOOLEAN DEFAULT FALSE
);
```

### 3. Real-time Updates
**Polling Mechanism:**
```dart
Timer.periodic(Duration(seconds: 2), (timer) {
  _loadMessages(); // Fetch new messages every 2 seconds
});
```

**Message Retrieval:**
```dart
Future<List<Map<String, dynamic>>> getMessages(int userId1, int userId2) async {
  // Fetches messages between two users
  // Orders by timestamp
  // Marks messages as read
}
```

### 4. Chat Access Points
**From Client Dashboard:**
- Client can chat with approved CAs/BLOs
- Navigation via contact buttons

**From CA/BLO Dashboard:**
- Access to client conversations
- Integrated with request approval system

---

## Notification System

### 1. Firebase Cloud Messaging (FCM)
**File: `fcm_utils.py`**

**Key Functions:**
- **`send_notification()`**: Sends push notifications
- **`save_fcm_token()`**: Stores device tokens
- **`get_user_fcm_token()`**: Retrieves user tokens

### 2. Notification Triggers

**Request Status Changes:**
```python
# When CA approves/rejects request
async def update_request_status(request_id, status):
    # Update database
    # Get client FCM token
    client_token = get_user_fcm_token(client_id)
    
    # Send notification
    if status == 'approved':
        send_notification(
            token=client_token,
            title="Request Approved!",
            body="Your CA request has been approved"
        )
    elif status == 'rejected':
        send_notification(
            token=client_token,
            title="Request Rejected",
            body="Your CA request has been rejected"
        )
```

**New Message Notifications:**
```python
# When user sends message
async def send_message(sender_id, receiver_id, message):
    # Save message to database
    # Get receiver FCM token
    receiver_token = get_user_fcm_token(receiver_id)
    
    # Send notification
    send_notification(
        token=receiver_token,
        title="New Message",
        body=f"You have a new message: {message[:50]}..."
    )
```

### 3. FCM Token Management
**Client Side (Flutter):**
```dart
// Get FCM token when app starts
FirebaseMessaging.instance.getToken().then((token) {
  // Send token to backend
  apiService.saveFCMToken(userId, token);
});
```

**Backend Storage:**
```sql
CREATE TABLE fcm_tokens (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    user_type VARCHAR(20), -- 'client', 'ca', 'blo', 'fp'
    fcm_token TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4. Notification Display
**In-App Notifications:**
- AppBar notification icon with badge count
- Notification list showing recent updates

**Push Notifications:**
- System-level notifications when app is closed
- Deep linking to relevant screens

---

## Database Schema

### Core Tables

**1. Users Table Structure:**
```sql
-- Clients
CREATE TABLE clients (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(255),
    email VARCHAR(255) UNIQUE,
    password_hash TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Chartered Accountants
CREATE TABLE chartered_accountants (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(255),
    email VARCHAR(255) UNIQUE,
    qualification VARCHAR(255),
    experience VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Bank Loan Officers
CREATE TABLE bank_loan_officers (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(255),
    email VARCHAR(255) UNIQUE,
    qualification VARCHAR(255),
    experience VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**2. Request Management:**
```sql
-- CA Requests
CREATE TABLE ca_requests (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    ca_id INTEGER REFERENCES chartered_accountants(id),
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- BLO Requests (similar structure)
CREATE TABLE blo_requests (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    blo_id INTEGER REFERENCES bank_loan_officers(id),
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**3. Communication System:**
```sql
-- Messages
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    sender_id INTEGER,
    receiver_id INTEGER,
    sender_type VARCHAR(20),
    receiver_type VARCHAR(20),
    message_text TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_read BOOLEAN DEFAULT FALSE
);

-- FCM Tokens
CREATE TABLE fcm_tokens (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    user_type VARCHAR(20),
    fcm_token TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## API Endpoints Summary

### Request Management
- `POST /send-request-to-ca` - Client sends request to CA
- `POST /send-request-to-blo` - Client sends request to BLO
- `PUT /update-request-status/{id}` - CA/BLO updates request status
- `GET /ca-requests/{ca_id}` - Get requests for specific CA
- `GET /blo-requests/{blo_id}` - Get requests for specific BLO

### Chat System
- `POST /send-message` - Send new message
- `GET /messages/{user1_id}/{user2_id}` - Get conversation messages
- `PUT /mark-messages-read` - Mark messages as read

### Notifications
- `POST /save-fcm-token` - Save user's FCM token
- `POST /send-notification` - Send push notification
- `GET /notifications/{user_id}` - Get user notifications

---

## Security Considerations

### Authentication
- JWT tokens for API authentication
- Secure storage of tokens using `flutter_secure_storage`
- Session management and token refresh

### Data Validation
- Input sanitization for all user inputs
- SQL injection prevention using parameterized queries
- Rate limiting on API endpoints

### Privacy
- Message encryption in transit
- Secure FCM token handling
- GDPR compliance for data deletion

---

This documentation provides a comprehensive overview of how the TaxMate system handles client requests, CA approvals/rejections, chat functionality, and notifications. The system uses a combination of REST APIs, real-time polling, and push notifications to provide a seamless user experience across all user types (Clients, CAs, BLOs, and FPs).
