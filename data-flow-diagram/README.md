![image png](https://github.com/user-attachments/assets/76ec3cfa-621d-4b23-821b-d0b91a478ff8)
## 📊 Data Flow Diagram (DFD)

The core of the platform is modeled around a centralized **Database** which branches into multiple domain-specific databases:

###  Inputs
- **Guests** can search properties, book stays, make payments, and leave reviews.
- **Hosts** can manage their listings and respond to reviews.
- **Admins** handle user management, logs, and data governance.

###  Core Modules
| Module                | Description |
|------------------------|-------------|
| `Payment`             | Handles all guest and host transactions securely. |
| `Searching and Booking` | Manages availability queries, bookings, and cancellations. |
| `User Management`     | Admin-level tool for managing platform users. |
| `Properties`          | Allows hosts to add/edit listings. |
| `Reviews`             | Facilitates guest reviews and host responses. |

###  Database Structure
The centralized database routes and stores information in the following dedicated DBs:

- `User DB`: Stores user profiles and credentials.
- `Listing DB`: Manages property details.
- `Booking DB`: Contains booking records and availability status.
- `Payment DB`: Secure ledger of all financial transactions.
- `Review DB`: Guest reviews and ratings.
- `Admin Logs DB`: Tracks administrative actions and system events.

---


