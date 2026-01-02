# Hindi Coaching Management System - Complete Development Instructions

## Project Overview

### Business Context
A Hindi language tutor conducts online coaching classes via Zoom. She currently faces several operational challenges that need automation through a web application.

### Current Pain Points

1. **Manual Attendance Tracking**
   - After each Zoom class, she manually notes down who attended
   - Time-consuming and error-prone
   - No centralized attendance record
   - Difficult to track attendance history per student

2. **Payment Collection Delays**
   - Students pay after every 4 classes
   - She manually tracks class counts to know when payment is due
   - Often forgets who has attended how many classes
   - No automated reminders, leading to payment delays
   - Manual calculation of dues

3. **Student Inquiry Management**
   - Receives inquiries via WhatsApp, email, phone calls
   - No centralized system to track potential students
   - Loses track of follow-ups

4. **No Student Portal**
   - Students have no way to check their attendance
   - Students cannot verify their payment status
   - Leads to disputes and confusion

### Proposed Solution

A comprehensive web application with three interfaces:

1. **Public Landing Page**
   - Showcases coaching services
   - Displays 2-3 sample class videos
   - Inquiry form for potential students
   - Professional presentation

2. **Student Portal**
   - Passwordless login (magic link via email)
   - Dashboard showing:
     - Total classes attended
     - Payment status (paid/pending)
     - Classes remaining until next payment
   - Complete attendance history
   - Payment history

3. **Admin Dashboard** (for the tutor)
   - View and manage all student inquiries
   - Add/edit/deactivate students
   - View attendance for all students
   - Automatic attendance marking via Zoom integration
   - Manual attendance override capability
   - Record payments
   - View students who need payment reminders
   - Batch management (different class timings)

4. **Automation Features**
   - **Zoom Integration**: Automatically fetch meeting participants after each class
   - **Auto-mark Attendance**: Match Zoom participants to student records and mark attendance
   - **Payment Reminders**: Automatically send email reminders when student completes 4 unpaid classes
   - **Email Notifications**: Welcome emails, inquiry confirmations, payment receipts

---

## Technical Requirements

### Must-Have Features (MVP - Phase 1)

#### Public Features
- Landing page with hero section, sample videos, inquiry form
- Inquiry form submission saves to database
- Email notification to admin on new inquiry
- Auto-reply email to student confirming inquiry received

#### Student Features
- Magic link authentication (passwordless)
- Student dashboard showing:
  - Total classes attended
  - Classes attended since last payment
  - Payment status
  - Next payment due date/amount
- Attendance history view (table/calendar)
- Payment history view

#### Admin Features
- Admin login (email/password)
- Dashboard showing:
  - Total students
  - Today's class attendance
  - Students with pending payments
  - Recent inquiries
- **Inquiry Management**:
  - View all inquiries
  - Filter by status (NEW, CONTACTED, ENROLLED, REJECTED)
  - Convert inquiry to student
  - Mark inquiry status
- **Student Management**:
  - Add new student (name, email, phone, batch, roll number)
  - Edit student details
  - Deactivate student
  - Assign student to batch
  - Send welcome email with login link
- **Batch Management**:
  - Create batch (name, schedule, Zoom meeting ID, price per 4 classes)
  - Edit batch details
  - View students in batch
- **Attendance Management**:
  - View attendance for specific date/batch
  - Manually mark/unmark attendance
  - Override auto-marked attendance
  - Trigger Zoom participant fetch
  - Review unmatched Zoom participants
- **Payment Management**:
  - Record payment for student
  - Mark payment method (Cash/UPI/Bank Transfer)
  - View payment history
  - Calculate classes covered by payment

#### Automation Features
- **Zoom Integration**:
  - Connect to Zoom API (Server-to-Server OAuth)
  - Fetch past meeting participants
  - After class ends, automatically fetch participant list
  - Match participants to students by name/roll number
  - Auto-mark attendance for matched students
  - Show unmatched participants for manual review
- **Email Automation**:
  - Send inquiry confirmation to student
  - Send inquiry notification to admin
  - Send welcome email when student is enrolled
  - Send payment reminder when student reaches 4 unpaid classes
  - Send magic link for student login
- **Scheduled Jobs** (Cron):
  - Daily check: Find students with 4+ unpaid classes
  - Send payment reminder emails
  - (Optional) Fetch Zoom attendance for classes in last 24 hours

---

## Technical Stack

### Core Framework
- **Next.js 14** (App Router)
  - Why: Full-stack framework, React-based, excellent DX
  - Server-side rendering for landing page
  - API routes for backend
  - File-based routing
  - Built-in optimization

### Frontend
- **React 18+**
  - Component-based UI
  - Hooks for state management
- **TypeScript**
  - Type safety
  - Better developer experience
  - Catch errors at compile time
- **Tailwind CSS**
  - Utility-first CSS
  - Fast styling
  - Responsive by default
  - No external CSS files needed

### Backend/API
- **Next.js API Routes**
  - Server-side logic
  - RESTful endpoints
  - TypeScript support
- **Server Actions** (Next.js 14)
  - For form submissions
  - Direct database mutations
  - No separate API layer needed for simple operations

### Database
- **PostgreSQL**
  - Relational database
  - Handles complex queries
  - ACID compliant
  - Excellent for structured data
- **Prisma ORM**
  - Type-safe database client
  - Auto-generated types from schema
  - Migration system
  - Excellent TypeScript integration
  - Prisma Studio for database visualization

### Authentication
- **NextAuth.js v4**
  - Email magic links (passwordless)
  - Credentials login for admin
  - Session management
  - Built-in security
  - JWT tokens
  - CSRF protection

### Email Service
- **Resend**
  - Modern email API
  - 3,000 emails/month free
  - React email templates
  - Excellent deliverability
  - Simple API

### External Integrations
- **Zoom API**
  - Server-to-Server OAuth
  - Past Meeting Participants endpoint
  - Meeting Details endpoint
  - Webhooks for real-time updates (optional)

### Hosting & Infrastructure
- **Vercel** (Free tier)
  - Next.js optimized hosting
  - Automatic deployments from Git
  - Serverless functions
  - Edge network
  - 100GB bandwidth/month free
- **Supabase** (Free tier)
  - Managed PostgreSQL
  - 500MB database
  - Automatic backups
  - Connection pooling
  - Direct database access

### Development Tools
- **ESLint**: Code linting
- **Prettier**: Code formatting
- **Git**: Version control
- **GitHub**: Repository hosting

### Optional Enhancements
- **Cron Jobs**: Vercel Cron or external service (cron-job.org)
- **Analytics**: Vercel Analytics or Google Analytics
- **Error Tracking**: Sentry (free tier)

---

## Database Schema

### Tables and Relationships

```prisma
// Prisma Schema

model Inquiry {
  id        String   @id @default(cuid())
  name      String
  email     String
  query     String   @db.Text
  createdAt DateTime @default(now())
  status    String   @default("NEW") // "NEW", "CONTACTED", "ENROLLED", "REJECTED"
}

model Student {
  id        String       @id @default(cuid())
  email     String       @unique
  name      String
  rollNo    String       @unique  // Used to match Zoom participants
  phone     String?
  batchId   String?
  joinedAt  DateTime     @default(now())
  isActive  Boolean      @default(true)
  
  batch      Batch?       @relation(fields: [batchId], references: [id])
  attendance Attendance[]
  payments   Payment[]
  
  @@index([email])
  @@index([rollNo])
}

model Batch {
  id                  String       @id @default(cuid())
  name                String       // e.g., "Monday 5 PM Batch"
  schedule            String       // e.g., "MON,WED 17:00"
  zoomMeetingId       String       @unique  // Recurring meeting ID
  zoomPassword        String?
  pricePerFourClasses Decimal      @db.Decimal(10, 2)
  isActive            Boolean      @default(true)
  createdAt           DateTime     @default(now())
  
  students   Student[]
  attendance Attendance[]
}

model Attendance {
  id                  String   @id @default(cuid())
  studentId           String
  batchId             String
  classDate           DateTime  // Date of the class
  zoomParticipantName String?   // Original name from Zoom
  markedBy            String    // "AUTO" or admin email
  createdAt           DateTime  @default(now())
  
  student Student @relation(fields: [studentId], references: [id], onDelete: Cascade)
  batch   Batch   @relation(fields: [batchId], references: [id], onDelete: Cascade)
  
  @@unique([studentId, classDate])  // One attendance per student per day
  @@index([studentId])
  @@index([batchId])
  @@index([classDate])
}

model Payment {
  id             String   @id @default(cuid())
  studentId      String
  amount         Decimal  @db.Decimal(10, 2)
  classesCovered Int      // Number of classes this payment covers (usually 4)
  paymentDate    DateTime @default(now())
  status         String   @default("CONFIRMED") // "PENDING", "CONFIRMED"
  method         String?  // "CASH", "UPI", "BANK_TRANSFER"
  notes          String?  @db.Text
  createdAt      DateTime @default(now())
  
  student Student @relation(fields: [studentId], references: [id], onDelete: Cascade)
  
  @@index([studentId])
  @@index([paymentDate])
}

model Admin {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  password  String   // hashed with bcrypt
  createdAt DateTime @default(now())
}
```

### Key Relationships

1. **Student → Batch**: Many-to-one (many students in one batch)
2. **Student → Attendance**: One-to-many (one student has many attendance records)
3. **Student → Payment**: One-to-many (one student has many payments)
4. **Batch → Attendance**: One-to-many (one batch has many attendance records)

### Indexes

Critical indexes for performance:
- Student email (for login)
- Student rollNo (for Zoom matching)
- Attendance studentId, batchId, classDate (for queries)
- Payment studentId, paymentDate (for queries)

---

## Application Architecture

### Directory Structure

```
hindi-coaching-app/
├── app/
│   ├── (public)/              # Public routes (no auth required)
│   │   ├── page.tsx           # Landing page
│   │   └── login/
│   │       └── page.tsx       # Student login page
│   ├── (student)/             # Student routes (require student auth)
│   │   ├── layout.tsx         # Student layout with navigation
│   │   ├── dashboard/
│   │   │   └── page.tsx       # Student dashboard
│   │   ├── attendance/
│   │   │   └── page.tsx       # Attendance history
│   │   └── payments/
│   │       └── page.tsx       # Payment history
│   ├── (admin)/               # Admin routes (require admin auth)
│   │   ├── layout.tsx         # Admin layout with sidebar
│   │   ├── dashboard/
│   │   │   └── page.tsx       # Admin dashboard
│   │   ├── inquiries/
│   │   │   └── page.tsx       # Manage inquiries
│   │   ├── students/
│   │   │   ├── page.tsx       # List students
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx   # Edit student
│   │   │   └── new/
│   │   │       └── page.tsx   # Add student
│   │   ├── batches/
│   │   │   └── page.tsx       # Manage batches
│   │   ├── attendance/
│   │   │   └── page.tsx       # Review attendance
│   │   └── payments/
│   │       └── page.tsx       # Record payments
│   ├── api/
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   │       └── route.ts   # NextAuth configuration
│   │   ├── inquiries/
│   │   │   └── route.ts       # Inquiry CRUD
│   │   ├── students/
│   │   │   └── route.ts       # Student CRUD
│   │   ├── attendance/
│   │   │   └── route.ts       # Attendance CRUD
│   │   ├── payments/
│   │   │   └── route.ts       # Payment CRUD
│   │   ├── zoom/
│   │   │   ├── fetch-participants/
│   │   │   │   └── route.ts   # Fetch Zoom participants
│   │   │   └── webhook/
│   │   │       └── route.ts   # Zoom webhook handler
│   │   └── cron/
│   │       └── payment-reminders/
│   │           └── route.ts   # Check and send reminders
│   ├── globals.css            # Global styles + Tailwind
│   └── layout.tsx             # Root layout
├── components/
│   ├── ui/                    # Reusable UI components
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Table.tsx
│   │   ├── Card.tsx
│   │   └── Modal.tsx
│   ├── landing/               # Landing page components
│   │   ├── Hero.tsx
│   │   ├── VideoSection.tsx
│   │   └── InquiryForm.tsx
│   ├── student/               # Student-specific components
│   │   ├── AttendanceTable.tsx
│   │   ├── PaymentCard.tsx
│   │   └── StatsCard.tsx
│   └── admin/                 # Admin-specific components
│       ├── StudentTable.tsx
│       ├── AttendanceReview.tsx
│       └── Sidebar.tsx
├── lib/
│   ├── prisma.ts              # Prisma client singleton
│   ├── auth.ts                # NextAuth configuration
│   ├── email.ts               # Email utilities (Resend)
│   ├── zoom.ts                # Zoom API utilities
│   ├── utils.ts               # Helper functions
│   └── validations.ts         # Zod schemas
├── prisma/
│   └── schema.prisma          # Database schema
├── public/
│   ├── images/                # Static images
│   └── videos/                # Or video placeholders
├── scripts/
│   └── create-admin.ts        # Script to create admin user
├── .env.example               # Environment variables template
├── .env                       # Actual environment variables (gitignored)
├── .gitignore
├── next.config.js
├── package.json
├── postcss.config.js
├── tailwind.config.js
├── tsconfig.json
└── README.md
```

### Route Groups Explanation

Next.js App Router uses folder names in parentheses `()` for route grouping:

- **(public)**: Routes accessible without authentication
- **(student)**: Routes requiring student authentication
- **(admin)**: Routes requiring admin authentication

Each group can have its own layout, allowing different navigation/styling.

---

## Core Functionality Specifications

### 1. Landing Page

**Route**: `/`

**Features**:
- Professional hero section with:
  - Headline: "Learn Hindi from Expert Tutors"
  - Subheadline describing the service
  - Call-to-action button scrolling to inquiry form
  - "Student Login" link in header
- Sample class videos section:
  - 3 video cards
  - Each with embedded YouTube video (or placeholder)
  - Title and description below each video
- Inquiry form section:
  - Fields: Name, Email, Query (textarea)
  - Submit button
  - Success/error message display
- Footer with copyright

**Technical Requirements**:
- Server-side rendered for SEO
- Form validation (Zod schema)
- On submit:
  - POST to `/api/inquiries`
  - Save to database
  - Send email to admin
  - Send auto-reply to student
  - Show success message
  - Clear form

### 2. Student Authentication

**Route**: `/login`

**Flow**:
1. Student enters email
2. System checks if email exists in Student table
3. If exists:
   - Generate magic link token
   - Send email with login link
   - Show "Check your email" message
4. If not exists:
   - Show "Email not found. Please contact admin."
5. Student clicks link in email
6. Token verified
7. Session created
8. Redirect to `/dashboard`

**Technical Requirements**:
- Use NextAuth.js Email provider
- Magic link expires in 24 hours
- Email template with branded link
- Session stored in JWT
- Middleware protects student routes

### 3. Student Dashboard

**Route**: `/dashboard`

**Display**:
- Welcome message with student name
- Stats cards showing:
  - Total classes attended
  - Classes since last payment
  - Payment status (Paid up / ₹X pending)
  - Next class date/time
- Quick links to attendance and payment history

**Data Sources**:
- Query Attendance table for total count
- Query Payment table for last payment
- Calculate unpaid classes
- Get batch schedule for next class

**Technical Requirements**:
- Server component fetching data
- Protected route (require auth)
- Display loading state
- Handle no data scenarios

### 4. Attendance History

**Route**: `/attendance`

**Display**:
- Table showing:
  - Date
  - Batch name
  - Status (Present)
  - Time (from Zoom data)
- Filter by month/year
- Sort by date (newest first)
- Pagination (20 per page)
- Export to PDF button (optional)

**Technical Requirements**:
- Server component with search params
- Query Attendance table filtered by studentId
- Join with Batch table for batch name
- Format dates nicely
- Handle empty state

### 5. Payment History

**Route**: `/payments`

**Display**:
- Table showing:
  - Date
  - Amount
  - Classes covered
  - Method
  - Status
- Total amount paid
- Next payment due calculation

**Technical Requirements**:
- Query Payment table filtered by studentId
- Sum total payments
- Calculate unpaid classes
- Show when next payment is due

### 6. Admin Dashboard

**Route**: `/admin/dashboard`

**Display**:
- Overview stats:
  - Total active students
  - Students with pending payments
  - New inquiries today
  - Today's classes scheduled
- Recent activity feed:
  - Latest inquiries
  - Recent payments
  - Recent attendance
- Quick actions:
  - Add student
  - Record payment
  - View today's attendance

**Technical Requirements**:
- Aggregate queries for stats
- Real-time or cached data
- Protected route (admin only)
- Fast loading with optimized queries

### 7. Inquiry Management

**Route**: `/admin/inquiries`

**Features**:
- Table showing all inquiries:
  - Name, Email, Query, Date, Status
- Filter by status dropdown
- Actions per inquiry:
  - View full query (modal/expand)
  - Change status
  - Convert to student (opens add student form with pre-filled data)
  - Delete inquiry
- Search by name/email

**Technical Requirements**:
- Table component with sorting
- Modal for viewing full query
- Form for status change
- Confirmation before delete
- Search functionality (client or server)

### 8. Student Management

**Route**: `/admin/students`

**Features**:
- Table showing all students:
  - Roll No, Name, Email, Batch, Status, Classes Attended, Payment Status
- Add student button
- Actions per student:
  - Edit (modal or separate page)
  - Deactivate/Activate
  - View details
  - Send welcome email
- Search by name/email/roll number
- Filter by batch, active status

**Add/Edit Student Form**:
- Fields: Name, Email, Phone, Roll No, Batch (dropdown), Is Active
- Validation: Email unique, Roll No unique
- On submit:
  - Save to database
  - Send welcome email if new student
  - Show success message

**Technical Requirements**:
- Server-side pagination
- Form validation (Zod)
- Unique constraint handling
- Optimistic updates
- Email sending on create

### 9. Batch Management

**Route**: `/admin/batches`

**Features**:
- Table showing all batches:
  - Name, Schedule, Students Count, Zoom Meeting ID, Price
- Add batch button
- Actions per batch:
  - Edit
  - Deactivate/Activate
  - View students in batch
  - Copy Zoom link

**Add/Edit Batch Form**:
- Fields: Name, Schedule, Zoom Meeting ID, Zoom Password, Price per 4 Classes
- Validation: Zoom Meeting ID unique
- On submit: Save to database

**Technical Requirements**:
- Form validation
- Unique Zoom Meeting ID check
- Calculate student count

### 10. Attendance Management

**Route**: `/admin/attendance`

**Features**:
- Date picker to select class date
- Batch dropdown
- "Fetch from Zoom" button
- Table showing:
  - Student name, Roll No, Status (Present/Absent), Marked By, Time
- For each student:
  - Toggle Present/Absent
  - Manual override
- Unmatched Zoom participants section:
  - List of names from Zoom not matched to students
  - Manually assign to student
- Save changes button

**Fetch from Zoom Flow**:
1. Click "Fetch from Zoom"
2. Select batch (gets Zoom Meeting ID)
3. Call Zoom API to get past meeting participants for that meeting ID
4. Match participants to students by name/roll no
5. Auto-create attendance records for matched students
6. Display unmatched participants for manual review
7. Admin reviews and confirms/modifies
8. Save to database

**Technical Requirements**:
- Call `/api/zoom/fetch-participants`
- Fuzzy matching algorithm for names
- Conflict handling (duplicate matches)
- Optimistic UI updates
- Bulk save operation

### 11. Payment Recording

**Route**: `/admin/payments`

**Features**:
- Select student (dropdown with search)
- Student info display:
  - Classes attended since last payment
  - Amount due
- Payment form:
  - Amount (pre-filled)
  - Classes covered (pre-filled with 4)
  - Payment method (dropdown)
  - Date (default: today)
  - Notes (optional)
- Submit button
- Recent payments table

**On Submit**:
- Save payment to database
- Optionally send receipt email to student
- Update payment status
- Show success message
- Clear form

**Technical Requirements**:
- Calculate unpaid classes
- Auto-calculate amount based on batch price
- Form validation
- Optimistic update
- Email receipt (optional)

### 12. Zoom Integration

**Utilities** (`lib/zoom.ts`):

**Functions**:

```typescript
// Get OAuth access token
async function getZoomAccessToken(): Promise<string>

// Fetch participants for a past meeting
async function getZoomMeetingParticipants(meetingId: string): Promise<ZoomParticipant[]>

// Get meeting details
async function getZoomMeeting(meetingId: string): Promise<ZoomMeeting>

// Match participant name to student
function matchParticipantToStudent(
  participantName: string,
  students: Student[]
): Student | null
```

**Matching Algorithm**:
1. Check if participant name contains student's roll number
2. Check if participant name contains student's full name (case-insensitive)
3. Use fuzzy string matching (Levenshtein distance) with threshold
4. Return best match or null

**API Route**: `/api/zoom/fetch-participants`

**Input**:
- Meeting ID
- Class date

**Process**:
1. Get Zoom access token
2. Fetch past meeting participants
3. Get all students in batch for that meeting
4. For each participant:
   - Try to match to student
   - If match: Create attendance record
   - If no match: Add to unmatched list
5. Return:
   - Matched students with attendance
   - Unmatched participants

**Output**:
```json
{
  "matched": [
    { "studentId": "...", "studentName": "...", "participantName": "...", "duration": 3600 }
  ],
  "unmatched": [
    { "participantName": "Unknown User", "duration": 1800 }
  ]
}
```

### 13. Email System

**Email Templates** (`lib/email.ts`):

**Functions**:

```typescript
// Send inquiry notification to admin
async function sendInquiryNotification(inquiry: { name, email, query })

// Send auto-reply to student on inquiry
async function sendInquiryAutoReply(inquiry: { name, email })

// Send welcome email when student enrolled
async function sendWelcomeEmail(student: { name, email, batchName, zoomLink, price })

// Send magic link for login
async function sendMagicLink(email: string, url: string)

// Send payment reminder
async function sendPaymentReminder(student: { name, email, classesAttended, amountDue })

// Send payment receipt (optional)
async function sendPaymentReceipt(payment: { studentName, amount, date })
```

**Email Templates**:

Each function uses Resend's API to send HTML emails.

Example template structure:
```html
<h2>Heading</h2>
<p>Body text</p>
<a href="link">Call to action</a>
<hr />
<p>Footer</p>
```

### 14. Payment Reminder Automation

**Cron Route**: `/api/cron/payment-reminders`

**Frequency**: Daily (run at 9 AM)

**Process**:
1. Query all active students
2. For each student:
   - Count total classes attended
   - Count total classes paid for (sum of classesCovered from payments)
   - Calculate unpaid classes = attended - paid
   - If unpaid classes >= 4:
     - Get batch price
     - Calculate amount due
     - Send payment reminder email
     - Log reminder sent
3. Return summary

**Technical Requirements**:
- Vercel Cron job or external cron service
- Secure with API key or Vercel Cron secret
- Error handling and logging
- Rate limiting for emails
- Prevent duplicate reminders (track last sent)

**Optimization**:
- Add `lastReminderSent` field to Student model
- Only send if last reminder was >7 days ago
- Prevents spam

---

## API Endpoints

### Public Endpoints

| Method | Route | Description | Auth |
|--------|-------|-------------|------|
| POST | `/api/inquiries` | Submit inquiry form | None |
| GET | `/api/auth/signin` | Show login page | None |
| POST | `/api/auth/signin` | Send magic link | None |

### Student Endpoints

| Method | Route | Description | Auth |
|--------|-------|-------------|------|
| GET | `/api/student/dashboard` | Get dashboard data | Student |
| GET | `/api/student/attendance` | Get attendance history | Student |
| GET | `/api/student/payments` | Get payment history | Student |

### Admin Endpoints

| Method | Route | Description | Auth |
|--------|-------|-------------|------|
| GET | `/api/inquiries` | List inquiries | Admin |
| PATCH | `/api/inquiries/[id]` | Update inquiry status | Admin |
| DELETE | `/api/inquiries/[id]` | Delete inquiry | Admin |
| GET | `/api/students` | List students | Admin |
| POST | `/api/students` | Create student | Admin |
| PATCH | `/api/students/[id]` | Update student | Admin |
| DELETE | `/api/students/[id]` | Deactivate student | Admin |
| GET | `/api/batches` | List batches | Admin |
| POST | `/api/batches` | Create batch | Admin |
| PATCH | `/api/batches/[id]` | Update batch | Admin |
| GET | `/api/attendance` | Get attendance for date/batch | Admin |
| POST | `/api/attendance` | Mark attendance manually | Admin |
| PATCH | `/api/attendance/[id]` | Update attendance | Admin |
| POST | `/api/zoom/fetch-participants` | Fetch from Zoom | Admin |
| GET | `/api/payments` | List payments | Admin |
| POST | `/api/payments` | Record payment | Admin |

### Cron Endpoints

| Method | Route | Description | Auth |
|--------|-------|-------------|------|
| GET/POST | `/api/cron/payment-reminders` | Check and send reminders | Cron Secret |

---

## Environment Variables

```env
# Database
DATABASE_URL="postgresql://..."

# NextAuth
NEXTAUTH_SECRET="..." # Generate with: openssl rand -base64 32
NEXTAUTH_URL="http://localhost:3000" # Change for production

# Email (Resend)
RESEND_API_KEY="re_..."
ADMIN_EMAIL="admin@example.com"
FROM_EMAIL="noreply@example.com"

# Zoom API
ZOOM_ACCOUNT_ID="..."
ZOOM_CLIENT_ID="..."
ZOOM_CLIENT_SECRET="..."

# App
NEXT_PUBLIC_APP_NAME="Hindi Coaching"
NEXT_PUBLIC_APP_URL="http://localhost:3000"

# Cron (for payment reminders)
CRON_SECRET="..." # Random string for cron job auth
```

---

## User Flows

### Student Enrollment Flow

1. **Potential student visits landing page**
2. **Watches sample videos**
3. **Fills inquiry form** (name, email, query)
4. **System saves inquiry** to database
5. **Admin receives email** notification
6. **Student receives auto-reply** email
7. **Admin reviews inquiry** in admin panel
8. **Admin marks inquiry as "CONTACTED"**
9. **Admin decides to enroll student**
10. **Admin clicks "Convert to Student"**
11. **Add student form opens** with pre-filled name/email
12. **Admin fills remaining details** (roll no, batch, phone)
13. **Admin submits form**
14. **System creates student** record
15. **System sends welcome email** with login link
16. **Student clicks login link** in email
17. **Student enters email** on login page
18. **System sends magic link** to email
19. **Student clicks magic link**
20. **Student logged in**, redirected to dashboard

### Class Attendance Flow

1. **Tutor conducts Zoom class**
2. **Class ends**
3. **Tutor logs into admin panel**
4. **Navigates to Attendance page**
5. **Selects date and batch**
6. **Clicks "Fetch from Zoom"**
7. **System fetches Zoom participants**
8. **System matches participants to students**
9. **System auto-marks attendance** for matched students
10. **System shows unmatched participants**
11. **Admin reviews auto-marked attendance**
12. **Admin manually assigns unmatched participants** to students (if applicable)
13. **Admin manually overrides** any incorrect matches
14. **Admin clicks "Save"**
15. **Attendance saved** to database

### Payment Collection Flow

1. **Cron job runs daily**
2. **Checks all students** for unpaid classes >= 4
3. **Sends payment reminder emails**
4. **Student receives reminder**, makes payment
5. **Admin logs into admin panel**
6. **Navigates to Payments page**
7. **Selects student** from dropdown
8. **System shows** classes attended, amount due
9. **Admin fills payment form** (amount, method, date)
10. **Admin clicks "Record Payment"**
11. **System saves payment** to database
12. **System optionally sends receipt** to student
13. **Student's dashboard updates** showing payment status

---

## Security Considerations

### Authentication
- Use NextAuth.js for secure authentication
- Magic links expire in 24 hours
- Admin passwords hashed with bcrypt (min 10 rounds)
- JWT tokens for session management
- HttpOnly cookies to prevent XSS

### Authorization
- Middleware checks user role on protected routes
- Students can only access their own data
- Admin can access all data
- API routes validate user role before processing

### Input Validation
- All form inputs validated with Zod schemas
- SQL injection prevented by Prisma ORM
- XSS prevented by React (auto-escaping)
- CSRF protection via NextAuth

### API Security
- Rate limiting on email endpoints (prevent spam)
- Cron endpoints secured with secret token
- Zoom API credentials stored in env vars
- Database credentials in env vars (never committed)

### Data Privacy
- Sensitive data (passwords, emails) encrypted at rest
- HTTPS enforced in production
- No sensitive data in client-side code
- Logs don't contain passwords or tokens

---

## Performance Optimization

### Database
- Indexes on frequently queried columns
- Connection pooling (Supabase default)
- Pagination for large lists
- Aggregate queries for stats (avoid N+1)

### Frontend
- Server components for initial page load
- Client components only when needed
- Image optimization (Next.js Image)
- Code splitting (automatic with Next.js)
- Static generation for landing page

### API
- Edge functions for low latency
- Caching where appropriate
- Batch operations (e.g., bulk attendance save)
- Debounce search inputs

### Email
- Queue email sending (don't block requests)
- Rate limiting to prevent spam
- Retry failed emails

---

## Testing Strategy

### Unit Tests
- Utility functions (zoom matching, calculations)
- Validation schemas
- Email template rendering

### Integration Tests
- API routes
- Database operations
- Authentication flow

### E2E Tests (Optional)
- User flows (inquiry, login, mark attendance)
- Critical paths
- Payment recording

### Manual Testing Checklist
- [ ] Landing page loads
- [ ] Inquiry form submits and sends emails
- [ ] Student login works
- [ ] Student dashboard shows correct data
- [ ] Admin login works
- [ ] Admin can add student
- [ ] Admin can manage inquiries
- [ ] Zoom integration fetches participants
- [ ] Attendance auto-marking works
- [ ] Payment recording updates student status
- [ ] Payment reminders send correctly

---

## Deployment Checklist

### Pre-Deployment
- [ ] All environment variables configured
- [ ] Database schema pushed to production DB
- [ ] Admin user created
- [ ] Email templates tested
- [ ] Zoom API credentials verified
- [ ] Domain configured (if using custom domain)

### Deployment Steps
1. Push code to GitHub
2. Connect Vercel to GitHub repo
3. Add environment variables in Vercel
4. Deploy
5. Run database migrations (if any)
6. Test all functionality in production
7. Set up cron job for payment reminders

### Post-Deployment
- [ ] Landing page accessible
- [ ] Student login works
- [ ] Admin login works
- [ ] Emails sending correctly
- [ ] Zoom integration working
- [ ] Cron job scheduled
- [ ] Monitor error logs
- [ ] Set up monitoring/alerts

---

## Monitoring & Maintenance

### Logs to Monitor
- Failed email sends
- Zoom API errors
- Authentication failures
- Payment reminder cron job status

### Regular Tasks
- Weekly: Review error logs
- Monthly: Check database size (ensure under free tier)
- Monthly: Review payment reminder effectiveness
- Quarterly: Review student feedback

### Backup Strategy
- Supabase automatic daily backups
- Manual exports before major changes
- Export student/payment data monthly

---

## Future Enhancements (Optional)

### Phase 2 Features
- WhatsApp notifications (via Twilio/WhatsApp Business API)
- Student certificates on course completion
- Attendance analytics and reports
- Batch-wise performance tracking
- Automated Zoom link in welcome email

### Phase 3 Features
- Mobile app (React Native)
- Live class scheduling within app
- In-app payment collection (Razorpay integration)
- Student progress tracking
- Homework assignments and submission
- Video library for recorded classes

### Advanced Features
- AI-powered student performance insights
- Automated follow-up for inactive students
- Referral system for students
- Multi-language support
- Custom reports generation (PDF)

---

## Development Timeline Estimate

### Phase 1: Foundation (Week 1)
- [ ] Project setup
- [ ] Database schema
- [ ] Landing page
- [ ] Inquiry system
- [ ] Email integration
- **Deliverable**: Working landing page with inquiry form

### Phase 2: Authentication & Student Portal (Week 2)
- [ ] NextAuth setup
- [ ] Student login
- [ ] Student dashboard
- [ ] Attendance history page
- [ ] Payment history page
- **Deliverable**: Students can login and view their data

### Phase 3: Admin Panel - Part 1 (Week 3)
- [ ] Admin authentication
- [ ] Admin dashboard
- [ ] Inquiry management
- [ ] Student management
- [ ] Batch management
- **Deliverable**: Admin can manage students and inquiries

### Phase 4: Admin Panel - Part 2 (Week 4)
- [ ] Attendance management UI
- [ ] Payment recording UI
- [ ] Manual attendance marking
- **Deliverable**: Admin can record attendance and payments manually

### Phase 5: Zoom Integration (Week 5)
- [ ] Zoom API setup
- [ ] Fetch participants endpoint
- [ ] Participant matching algorithm
- [ ] Auto-mark attendance
- [ ] Unmatched participants handling
- **Deliverable**: Automated attendance from Zoom

### Phase 6: Automation & Polish (Week 6)
- [ ] Payment reminder cron job
- [ ] Email templates refinement
- [ ] UI/UX improvements
- [ ] Testing and bug fixes
- [ ] Documentation
- **Deliverable**: Fully automated system ready for production

### Total Estimated Time: 6 weeks (part-time) or 3 weeks (full-time)

---

## Critical Implementation Notes

### For Coding Agents

1. **Always use TypeScript** - No JavaScript files
2. **Use Server Components by default** - Only use Client Components when needed (forms, interactive UI)
3. **Environment variables must never be committed** - Always use .env.local and .env.example
4. **Prisma schema changes require migration** - Run `npx prisma generate` and `npx prisma db push`
5. **NextAuth requires NEXTAUTH_SECRET** - Generate securely
6. **Zoom API uses OAuth 2.0** - Token must be fetched before each API call
7. **Email sending should not block requests** - Consider async/background processing
8. **All forms need Zod validation** - Both client and server-side
9. **Dates in UTC** - Convert to local timezone for display
10. **Protect all admin routes** - Use middleware or getServerSession
11. **Student can only see their own data** - Filter queries by studentId from session
12. **Unique constraints** - Handle duplicate email/rollNo gracefully
13. **Soft delete** - Use isActive flag instead of actual deletion
14. **Pagination for large lists** - Don't fetch all records at once
15. **Error handling** - Always show user-friendly error messages

### Common Pitfalls to Avoid

1. **Don't store passwords in plain text** - Always hash with bcrypt
2. **Don't expose API keys in client code** - Use environment variables and server-side code
3. **Don't trust client input** - Validate on server
4. **Don't forget to handle edge cases** - Empty states, no data, errors
5. **Don't use synchronous operations in API routes** - Always async/await
6. **Don't hardcode values** - Use environment variables for configuration
7. **Don't skip error boundaries** - Wrap components to catch errors
8. **Don't forget loading states** - Show spinners/skeletons while fetching
9. **Don't make unnecessary database calls** - Use React Query or SWR for caching
10. **Don't ignore TypeScript errors** - Fix them, don't bypass with 'any'

---

## Code Quality Standards

### TypeScript
- Strict mode enabled
- No 'any' types (use 'unknown' if needed)
- Interfaces for complex objects
- Enums for status values

### Formatting
- Prettier for code formatting
- ESLint for linting
- Consistent naming conventions:
  - camelCase for variables/functions
  - PascalCase for components/types
  - UPPER_CASE for constants

### File Organization
- One component per file
- Group related files in folders
- Index files for clean imports
- Separate utils, types, constants

### Comments
- JSDoc for public functions
- Inline comments for complex logic
- TODO comments for future work
- No commented-out code (use git)

---

## Support Resources

### Documentation Links
- Next.js: https://nextjs.org/docs
- Prisma: https://www.prisma.io/docs
- NextAuth: https://next-auth.js.org
- Tailwind: https://tailwindcss.com/docs
- Resend: https://resend.com/docs
- Zoom API: https://developers.zoom.us/docs/api

### Community
- Next.js Discord
- Stack Overflow
- GitHub Issues

---

## Success Criteria

The project is considered complete when:

1. ✅ Landing page is live and functional
2. ✅ Inquiry form saves to database and sends emails
3. ✅ Students can login with magic links
4. ✅ Students can view their attendance and payment history
5. ✅ Admin can login and access admin panel
6. ✅ Admin can manage inquiries (view, update status, convert to student)
7. ✅ Admin can manage students (add, edit, deactivate)
8. ✅ Admin can manage batches (add, edit, view students)
9. ✅ Admin can fetch Zoom participants and auto-mark attendance
10. ✅ Admin can manually override attendance
11. ✅ Admin can record payments
12. ✅ Cron job sends payment reminders to students with 4+ unpaid classes
13. ✅ All features work in production (Vercel)
14. ✅ No critical bugs
15. ✅ All environment variables documented
16. ✅ README with setup instructions

---

## Final Notes for Coding Agent

This is a real-world application for an actual business. Prioritize:

1. **Reliability**: The system must work consistently
2. **Simplicity**: Keep code simple and maintainable
3. **Security**: Protect user data and credentials
4. **User Experience**: Both admin and students should find it easy to use
5. **Cost Efficiency**: Stay within free tiers

Build incrementally. Test each feature before moving to the next. Document as you go. Ask for clarification if requirements are unclear.

The tutor (your sister) is non-technical, so the admin interface must be intuitive and forgiving of mistakes (confirmations before destructive actions, undo functionality where possible).

Students may have varying levels of tech literacy, so the student portal must be extremely simple and self-explanatory.

Good luck! 🚀
