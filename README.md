# 🪙GoPay 
GoPay is a **mobile wallet system** that allows users to **store, send, and receive money** digitally.It provides a simple, secure, and fast way to handle financial transactions.  

------------------------------------------------------------------------------------------------------

## 1. Overview
GoPay enables:  
- Peer-to-Peer (P2P) money transfers.  
- Cash in/out via agents or bank integration.  
- Bill payments (electricity, water, internet, etc.).  
- Mobile recharge
- QR-based payments at merchants.    
- Remote Lock/Wipe in case of lost or stolen phone. 

------------------------------------------------------------------------------------------------------

## 2. Scope

### 👤 Users Features 
- Register & verify account (KYC).  
- Add money (cash in) & withdraw (cash out).  
- Send/receive money to other users.  
- Pay bills. 
- Mobile recharge 
- QR payments at merchants.  
- View transaction history & wallet balance.   
- Notifications for all transactions.  
- Remote lock/wipe in case of lost phone.  
- Profile management.   

### 🛠️ Admin Features 
- Users management.  
- Dashboard with analytics.  
- Manage agents (who allow cash in/out).  
- Manage billers & merchants.  
- Monitor suspicious transactions (fraud detection) (optional).  
- Handle system reports & logs.  
- System health monitoring.  
- Role management (Admin, Sub-admin, Auditor) (optional).   

------------------------------------------------------------------------------------------------------
## 3.🏗️ System Architecture  

### System Layers & Technologies

1. **Client Layer (Mobile/Web App)**
   - **Mobile App**: Flutter → iOS & Android
   - **Admin Dashboard (Web)**: React, Html, CSS, JS
   - Responsibilities:
     - UI for users and admin
     - Display wallet balance, transaction history
     - Handle P2P transfers, bill payments, mobile recharge

2. **Backend Layer (API / Server)**
   - **Python + Django**
   - **RabbitMQ + Celery** → Background tasks (notifications)
   - Responsibilities:
     - Business logic (transactions, P2P, Cash in/out, Bill Payment, Mobile Recharge)
     - Authentication & Authorization (JWT)
     - Logging & Error Handling

3. **Database Layer**
   - **PostgreSQL**
   - Responsibilities:
     - Store Users, Wallet balances, Transactions
     - Store Agents, Merchants, Billers
     - Audit logs, reports
     - Historical data for analytics / BI 

4. **Integration / Gateway Layer**
   - Bank API Gateway → Cash in/out via banks
   - Payment & Mobile Recharge APIs → Top-up
   - Billers API → electricity, water, internet
   - Responsibilities:
     - Communicate with external services 
     - Map external responses to internal models
   - Note: This layer is implemented as part of the backend modules

5. **DevOps / Deployment Layer**
   - **Docker** → Containerize backend 
   - **GitLab** → Version control & CI/CD
   - Responsibilities:
     - Deploy backend & web apps
     - Run CI/CD pipelines for updates


------------------------------------------------------------------------------------------------------
## 4.Sequence Flow Diagrams:
### User Interactions:

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant App as "Mobile App"
    participant API as "Backend API"
    participant DB as "Database"
    participant Notify as "Notification Service"
    participant Bank as "Bank API"
    participant Bill as "Billers API"
    participant Recharge as "Mobile Recharge API"

    %% 1. Registration & KYC
    User->>App: Register with details
    App->>API: Submit registration
    activate API
    API->>DB: Store user account
    DB-->>API: Success
    API->>API: Trigger KYC process
    API->>DB: Update user status (KYC)
    DB-->>API: KYC Verified
    API->>Notify: Send KYC completion to user
    Notify-->>User: KYC Success Notification
    deactivate API
    API-->>App: Registration/KYC result
    App-->>User: Show success/fail

    %% 2. Add Money (Cash-in)
    User->>App: Add money request
    App->>API: Add money (amount)
    activate API
    API->>Bank: Bank debit request
    Bank-->>API: Debit confirmation
    API->>DB: Update wallet balance
    DB-->>API: Updated balance
    API->>Notify: Notify user of cash-in
    Notify-->>User: Cash-in Success Notification
    deactivate API
    API-->>App: Show new balance
    App-->>User: Display balance

    %% 3. P2P Transfer
    User->>App: Send money to user
    App->>API: P2P transfer request (to, amount)
    activate API
    API->>DB: Check sender balance
    DB-->>API: Sufficient/insufficient
    alt Sufficient balance
        API->>DB: Debit sender, credit receiver
        DB-->>API: Transfer committed
        API->>Notify: Notify both users
        Notify-->>User: P2P Sent Notification
    else Insufficient
        API-->>App: Insufficient funds
    end
    deactivate API
    API-->>App: Transfer result
    App-->>User: Show transfer status

    %% 4. Pay Bill
    User->>App: Pay bill (details)
    App->>API: Bill payment request
    activate API
    API->>DB: Check wallet balance
    DB-->>API: Sufficient/insufficient
    alt Sufficient balance
        API->>Bill: Billers API payment
        Bill-->>API: Payment confirmation
        API->>DB: Deduct amount, log transaction
        DB-->>API: Updated balance
        API->>Notify: Bill payment notification
        Notify-->>User: Bill Paid Notification
    else Insufficient
        API-->>App: Insufficient funds
    end
    deactivate API
    API-->>App: Show payment status
    App-->>User: Display bill payment result

    %% 5. Mobile Recharge
    User->>App: Mobile recharge (number, amount)
    App->>API: Recharge request
    activate API
    API->>DB: Check wallet balance
    DB-->>API: Sufficient/insufficient
    alt Sufficient balance
        API->>Recharge: Mobile Recharge API
        Recharge-->>API: Recharge confirmation
        API->>DB: Deduct amount, log transaction
        DB-->>API: Updated balance
        API->>Notify: Mobile recharge notification
        Notify-->>User: Recharge Successful Notification
    else Insufficient
        API-->>App: Insufficient funds
    end
    deactivate API
    API-->>App: Show recharge status
    App-->>User: Display recharge result

    %% 6. View Wallet Balance & History
    User->>App: View balance/history
    App->>API: Fetch balance & transactions
    activate API
    API->>DB: Get balance & transaction history
    DB-->>API: Return data
    deactivate API
    API-->>App: Data result
    App-->>User: Display balance/history

    %% 7. Notifications for Transactions
    Note over Notify,User: All key transactions generate notifications to the user via the Notification Service
