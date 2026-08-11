**Second PRD**

# **Product Summary**

## **_What are we building?_**

Second is an AI-powered cognitive companion that helps people offload information from their minds with minimal effort. Instead of asking users to remember meetings, subscriptions, screenshots, reminders, ideas, or other important information, Second lets them save it directly from any app using the native Share menu.

The system automatically understands what has been shared, organizes it into the appropriate category, and makes it easy to retrieve when needed. Rather than replacing human memory, Second reduces unnecessary cognitive load so users can focus on what matters most.

# **Goals**

## **_What should the product achieve?_**

- Reduce the mental effort required to remember everyday information.
- Minimize the friction involved in saving information.
- Allow users to save information without interrupting their current task.
- Automatically organize information into meaningful categories.
- Make saved information easy to find through search and contextual reminders.
- Build trust by allowing users to review AI-generated information before saving.
- Fit naturally into existing smartphone workflows instead of replacing them.

# **Features**

## **_What functionality exists?_**

### **Core Features**

- Share → Second integration
- AI-powered information extraction
- Meeting detection
- Reminder creation
- Subscription tracking
- Memory Vault
- Screenshot organization
- Universal search
- Review Inbox
- Morning Brief
- Push notifications
- Calendar view
- User authentication
- Guest mode
- Settings and preferences

### **Supporting Features**

- AI confidence scoring
- Edit before saving
- Notification preferences
- Review Now / Review Later workflow
- Cross-device synchronization
- Secure cloud backup

# **Functional Requirements**

## **_What exactly must the system do?_**

### **Capture**

- Accept shared content from supported applications. (email, whatsapp, social media, notes)
- Detect the type of content automatically.
- Extract structured information using AI.
- Support text, images, URLs and documents.

### **Processing**

- Identify meetings, reminders, subscriptions and memories.
- Extract relevant metadata such as date, time, location, people and title.
- Assign confidence scores to extracted information.
- Request user confirmation for low-confidence extractions.

### **Storage**

- Save approved items securely.
- Organize information into the correct category.
- Synchronize data across devices for signed-in users.

### **Retrieval**

- Allow users to search using natural language.
- Display today's events and reminders.
- Surface upcoming subscriptions and pending reviews.
- Generate a daily morning summary.

### **Notifications**

- Send reminder notifications.
- Notify users of subscription renewals.
- Deliver Morning Brief notifications.
- Respect user notification preferences.

# **User Flows**

## **_How does the user interact with it?_**

### **Onboarding**

Launch App  
↓  
Introduction  
↓  
Create Account / Continue as Guest  
↓  
Notification Permission  
↓  
Share Sheet Tutorial  
↓  
Home Screen

### **Saving Information**

View Content  
↓  
Long Press  
↓  
Share  
↓  
Second  
↓  
AI Extraction  
↓  
Review  
↓  
Save  
↓  
Return to Original App

### **Review Inbox**

Home  
↓  
Review Inbox  
↓  
Review Pending Items  
↓  
Edit  
↓  
Save

###

### **Morning Brief**

Open App  
↓  
Today's Meetings  
↓  
Today's Reminders  
↓  
Subscription Renewals  
↓  
Memory Vault

### **Search**

Open Search  
↓  
Type Query  
↓  
AI Search  
↓  
Relevant Results

# **Tech Stack**

## **_What technologies will be used?_**

### **Frontend**

- Flutter
- Dart

### **Backend**

- ASP.NET Core (.NET 9 or latest LTS)
- C#

### **AI**

- OpenAI GPT
- OCR (Apple Vision / Google ML Kit / Azure Document Intelligence later if needed)
- Natural Language Processing

### **Database**

- PostgreSQL
- Entity Framework Core

### **Authentication**

- Firebase Authentication or ASP.NET Identity + JWT (more control)
- Apple Sign In
- Google Sign In

### **Cloud**

- AWS
- S3
- CloudFront

### **Notifications**

- Firebase Cloud Messaging
- Apple Push Notification Service

### **Search**

- PostgreSQL Full Text Search (MVP)
- Elasticsearch/OpenSearch (future)

### **Analytics**

- Firebase Analytics
- Crashlytics

# **Architecture**

## **_How will components connect?_**

Flutter App  
↓  
Native Share Extension  
↓  
Backend API  
↓  
AI Processing Engine  
↓  
Structured Data Extraction  
↓  
PostgreSQL Database  
↓  
Search Index  
↓  
Notification Service  
↓  
Flutter App

The mobile application receives shared content through the native Share Extension. The content is securely uploaded to the backend where AI extracts structured information. The processed data is stored in the database, indexed for search, and used to generate reminders and contextual notifications before being returned to the application.

#

# **File Formats**

## **_What inputs and outputs are supported?_**

### **Inputs**

- Plain text
- Rich text
- Images
- Screenshots
- PDF documents
- URLs
- Email content
- Calendar invitations

### **Outputs**

- Meetings
- Reminders
- Subscriptions
- Memories
- Searchable records
- Push notifications
- Morning Brief summaries

# **UI Requirements**

## **_What should it look like?_**

### **Design Language**

- Clean
- Minimal
- Calm
- Premium
- Apple-inspired

### **Theme**

- Light mode first
- Deep Teal (#0F766E) as the primary accent
- White backgrounds
- Soft gray surfaces
- High whitespace

### **Color Styles**

The following local paint styles from the Second Figma file are the source of truth for product colors:

| Style | Hex |
| --- | --- |
| Primary Accent | `#0F766E` |
| Primary Background | `#FFFFFF` |
| Secondary Background | `#F8FAFA` |
| Card Background | `#F2F7F7` |
| Primary Text | `#111827` |
| Secondary Text | `#6B7280` |
| Divider | `#E5E7EB` |
| Success | `#14B8A6` |
| Warning | `#F59E0B` |
| Error | `#DC2626` |
| Button Primary | `#0F766E` |
| Button Hover | `#0B5F59` |
| Button Pressed | `#084A46` |

Primary Accent is used for primary buttons, selected states, active tabs, toggles, progress indicators, success checkmarks, and Share → Second animations.

### **Typography**

- Large, bold headlines
- Simple body text
- High readability

### **Components**

- Rounded buttons
- Soft cards
- Native mobile interactions
- Bottom sheets
- Smooth transitions

### **Motion**

- Purposeful animations only
- Information flows naturally
- Minimal interruptions
- Fast interactions
- Subtle micro-interactions

### **User Experience**

- One primary action per screen
- Minimal typing
- Minimal setup
- AI always remains editable
- Users stay in control

# **Acceptance Criteria**

## **_How do we know it works?_**

### **Capture**

- Users can share supported content from any compatible application.
- Second appears in the native Share menu.
- Shared content reaches the backend successfully.

### **AI Extraction**

- Meetings are identified correctly.
- Reminders are extracted correctly.
- Subscriptions are recognized correctly.
- Users can edit extracted information before saving.

### **Storage**

- Saved items appear in the correct category.
- Data persists across app launches.
- Signed-in users receive cross-device synchronization.

### **Retrieval**

- Search returns relevant results.
- Morning Brief displays today's information correctly.
- Review Inbox shows pending items.

### **Notifications**

- Reminder notifications trigger at the correct time.
- Subscription reminders trigger before renewal.
- Notification preferences are respected.

### **Performance**

- Share processing completes within a few seconds.
- Home screen loads quickly.
- Search results appear almost instantly.
- The application remains responsive throughout all core workflows.

### **User Experience**

- Users can complete onboarding without assistance.
- First-time users successfully save their first item.
- Users can change preferences at any time.
- The complete Share → Review → Save workflow works without errors.
