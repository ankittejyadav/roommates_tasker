# Roommate Task Management App

## 1. Project Description

This project is a web application designed to help roommates manage shared household chores and responsibilities. The primary goal is to build and host this application at **zero cost**, using services with generous free tiers.

Key functions include assigning recurring chores (kitchen, bathroom cleaning), managing ad-hoc tasks (taking out trash), and allowing roommates to send real-time alerts for household issues (e.g., "sink is dirty").

## 2. Core Features

### Must-Haves

- **User Authentication**: Only roommates can sign up, log in, and access the application.
- **Recurring Task Assignment**: Ability to define and assign monthly recurring chores like "Clean Kitchen" and "Clean Bathroom" to different roommates.
- **Ad-Hoc Task Management**: A way to log and track one-off tasks like "Take out the trash."
- **Alert System**: A feature for any roommate to send an immediate notification/alert to the group (e.g., "Sink is full of dishes," "Too much hair in the bathroom drain").

### Good-to-Haves

- **Automated Chore Rotation**: Automatically rotate recurring monthly chores among all registered roommates.
- **Shared Shopping List**: A simple, collaborative list for household supplies.
- **Basic Expense Tracker**: A log for shared expenses (e.g., cleaning supplies) and who paid.
- **Shared Calendar**: To track events like when guests are over or when someone is out of town.

## 3. Technology Stack (The "Free Tier" Stack)

This stack is chosen specifically to ensure the project can be built, deployed, and maintained for $0.

- **Frontend**: **Next.js (React)**
  - **Why**: A powerful framework for building modern user interfaces. It integrates seamlessly with Vercel for hosting.
- **Backend & Database**: **Supabase**
  - **Why**: An all-in-one backend-as-a-service (BaaS) that provides a free-tier PostgreSQL database, built-in user authentication, and serverless edge functions. This single tool handles our entire backend.
- **Hosting**: **Vercel**
  - **Why**: Offers a seamless, Git-based deployment experience for Next.js apps with a generous free "Hobby" plan.
- **Styling**: **Tailwind CSS**
  - **Why**: A utility-first CSS framework for rapid UI development. It's configured by default with the `create-next-app` installer.
- **Email Notifications**: **Resend**
  - **Why**: An email API with a free tier for sending notification emails (e.g., task reminders, new alerts).

## 4. Database Schema (Supabase PostgreSQL)

You will need to create the following tables in the Supabase Table Editor.

### Table: `users`

- This table is automatically managed by **Supabase Auth**. It will store user information like `id`, `email`, etc.

### Table: `tasks`

- This table defines the chores that exist in the household.
- **Columns**:
  - `id` (uuid, primary key, default: `gen_random_uuid()`)
  - `name` (text, not null)
  - `description` (text, nullable)
  - `frequency` (text, e.g., "monthly", "weekly", "once")
  - `created_at` (timestamp, default: `now()`)

### Table: `assignments`

- This table links a user to a task for a specific period.
- **Columns**:
  - `id` (uuid, primary key, default: `gen_random_uuid()`)
  - `task_id` (uuid, foreign key, references `tasks.id`)
  - `user_id` (uuid, foreign key, references `auth.users.id`)
  - `due_date` (timestamp)
  - `is_completed` (boolean, default: `false`)
  - `created_at` (timestamp, default: `now()`)

### Table: `alerts`

- This table logs the ad-hoc notifications sent by roommates.
- **Columns**:
  - `id` (uuid, primary key, default: `gen_random_uuid()`)
  - `message` (text, not null)
  - `created_by` (uuid, foreign key, references `auth.users.id`)
  - `created_at` (timestamp, default: `now()`)

## 5. Step-by-Step Build Plan

### Step 1: Project Setup

1.  **Initialize Next.js**:
    ```bash
    npx create-next-app@latest roommate-task-app --typescript --tailwind --eslint
    cd roommate-task-app
    ```
2.  **Install Supabase Client**:
    ```bash
    npm install @supabase/supabase-js
    ```
3.  **Set Up Supabase Project**:
    - Create a new free-tier project at [supabase.com](https://supabase.com).
    - Navigate to "Project Settings" > "API".
    - Find your **Project URL** and `anon` **public key**.
4.  **Configure Environment Variables**:
    - Create a file named `.env.local` in the root of your Next.js project.
    - Add your Supabase keys:
      ```
      NEXT_PUBLIC_SUPABASE_URL=YOUR_SUPABASE_PROJECT_URL
      NEXT_PUBLIC_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
      ```

### Step 2: Implement Authentication

1.  **Use Supabase Auth UI**: Install the official Supabase auth helpers for Next.js to easily create login, sign-up, and password-reset pages.
    ```bash
    npm install @supabase/auth-helpers-nextjs
    ```
2.  **Create Login/Sign-up Pages**: Build components for `/login` and `/signup`. Use the `supabase.auth.signInWithPassword()` and `supabase.auth.signUp()` methods.
3.  **Protect Routes**: Implement logic in your app (e.g., in a layout component or middleware) to check if a user is logged in. If not, redirect them to the `/login` page.

### Step 3: Build the Core App UI & Logic

1.  **Create Database Tables**: Go to the Supabase dashboard and use the Table Editor to create the `tasks`, `assignments`, and `alerts` tables as defined in the **Database Schema** section.
2.  **Task Dashboard (Read)**:
    - Create a main dashboard page (`/`).
    - On this page, fetch and display data.
    - Show "My Tasks": A list of tasks from the `assignments` table where `user_id` matches the logged-in user and `is_completed` is `false`.
    - Show "All Tasks": A view of all assignments for the current month.
3.  **Task Management (Create/Update)**:
    - Build a "Mark as Complete" button for each task, which updates the `is_completed` flag in the `assignments` table.
    - Build a simple form to manually create and assign a new task (e.g., for "Take out the trash"). This form will insert a new row into the `assignments` table.
4.  **Alert System (Create/Read)**:
    - Create a component with a text input and a "Send Alert" button.
    - When clicked, this button inserts a new row into the `alerts` table with the `message` and the `created_by` (current user's ID).
    - Display a feed of recent alerts on the dashboard by fetching data from the `alerts` table.

### Step 4: Automate Recurring Chores

1.  **Create a Supabase Edge Function**:
    - In your Supabase project, go to "Edge Functions".
    - Create a new function (e.g., `assign-monthly-chores`).
    - Write Deno/TypeScript code for this function. The logic should:
      1.  Fetch all users.
      2.  Fetch all "monthly" tasks from the `tasks` table.
      3.  Implement a rotation logic (e.g., based on who did it last time, or a simple round-robin).
      4.  For each monthly task, create a new entry in the `assignments` table, assigning it to the next user in the rotation with a `due_date` of (e.g.) the end of the new month.
2.  **Schedule a Cron Job**:
    - Use Supabase's built-in "Cron Jobs" feature (under `pg_cron` in the database settings or via the dashboard) to run your `assign-monthly-chores` function automatically on the first day of every month.

### Step 5: Implement Email Notifications

1.  **Set Up Resend**:
    - Sign up for a free Resend account.
    - Verify your domain (or use the test domain for development).
    - Get your Resend API key.
    - Store this key securely in your Supabase project's "Secrets" (under Edge Functions), _not_ in your Next.js code. Let's call it `RESEND_API_KEY`.
2.  **Create a Notification Edge Function**:
    - Create a new Supabase Edge Function (e.g., `send-alert-email`).
    - This function will be triggered by your database.
    - The function should:
      1.  Receive the new `alert` or `assignment` data as a payload.
      2.  Fetch all user emails from the `auth.users` table.
      3.  Use the `RESEND_API_KEY` (from secrets) to make a `fetch` request to the Resend API, sending an email to all users with the alert message.
3.  **Create Database Triggers**:
    - Go to "Database" > "Triggers" in your Supabase dashboard.
    - Create a new trigger on the `alerts` table.
    - Set it to fire `AFTER INSERT`.
    - Have this trigger call your `send-alert-email` Edge Function, passing it the new row's data.
    - (Optional) Create a similar trigger for the `assignments` table to notify a user when they get a new task.

### Step 6: Deploy to Vercel

1.  **Push to GitHub**: Create a new, free repository on GitHub and push your Next.js project code.
2.  **Import to Vercel**:
    - Sign up for a free Vercel account with your GitHub profile.
    - Select "Add New... Project" and import your new GitHub repository.
3.  **Configure Vercel**:
    - Vercel will auto-detect it's a Next.js project.
    - Go to the project's "Settings" > "Environment Variables".
    - Add your Supabase keys. **Important**: These MUST match the ones in your `.env.local` file.
      - `NEXT_PUBLIC_SUPABASE_URL`
      - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
4.  **Deploy**: Click "Deploy". Vercel will build and host your app on a public URL (e.g., `roommate-task-app.vercel.app`). Your roommates can now access it from any device.

## 6. Environment Variables Summary

You will need the following environment variables:

**In `.env.local` (for Next.js local development):**
