# Project Setup: Getting Started with Bump

## Overview
This guide provides a high-level overview for setting up the Bump application and understanding its structure. For detailed implementation steps, refer to the milestone-specific guides.

## Quick Start

```bash
# 1. Create a new Vite project with React and TypeScript
npm create vite@latest bump --template react-ts
cd bump

# 2. Install core dependencies
npm install react react-dom react-router-dom styled-components @react-google-maps/api @supabase/supabase-js workbox-window
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom typescript vite @vitejs/plugin-react vite-plugin-pwa nodemailer

# 3. Setup Supabase locally (if not already installed)
# Check if Supabase CLI is available
if ! command -v supabase &> /dev/null; then
    echo "Supabase CLI not found. Installing via Homebrew..."
    brew install supabase/tap/supabase
fi

# Initialize Supabase project
supabase init
supabase start
```

## Project Structure Overview

The Bump application follows this high-level structure:

```
bump/
├── .cursorguides/           # Implementation guides for each milestone
├── .cursorrules             # Project rules and guidelines
├── .cursortrack.md          # Project tracking document
├── public/                  # Static assets
├── src/
│   ├── components/          # Reusable UI components
│   ├── context/             # React context providers
│   ├── hooks/               # Custom React hooks
│   ├── pages/               # Route-level components
│   ├── styles/              # Theme and global styles
│   ├── types/               # TypeScript interfaces and types
│   ├── utils/               # Helper functions and utilities
│   ├── test/                # Test utilities and mocks
│   ├── App.tsx              # Main app component with routing
│   └── main.tsx             # Entry point
├── .env.development         # Development environment variables
├── .env.test                # Test environment variables
```

## Environment Configuration

Create these environment files to configure your local development:

1. `.env.development` - For local development environment:
```
# Supabase configuration
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0

# Email service for OTP (development only)
VITE_EMAIL_SERVICE=console
```

2. `.env.test` - For testing environment:
```
# Testing configuration with mock settings
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=test-key
VITE_EMAIL_SERVICE=mock
```

## Project Milestones

The Bump application is built through these milestones:

1. **Initialize Project** - Basic structure, routing, and styling system
2. **Supabase Integration** - Backend setup and authentication
3. **Home & Check-In** - Home page with status list and check-in
4. **Gathering Places** - Places page with list/map toggle
5. **Friends & Intent** - Friends page with connections and intent
6. **Meetups** - Meetup logging and history
7. **Settings** - User preferences
8. **Notifications** - Notification functionality
9. **Polish & Deploy** - Final touches and deployment

For detailed implementation steps for each milestone, refer to the corresponding guide in the `.cursorguides` directory.

## Key Principles

1. **Type Safety**: Use TypeScript with strict mode for better reliability.
2. **Component Reusability**: Create modular components that can be used across the app.
3. **Testing First**: Write tests before or alongside feature implementation.
4. **Theme Consistency**: Use the theme system for all styling to maintain visual consistency.
5. **Progressive Enhancement**: Build core functionality first, then enhance the user experience.

## Common Gotchas

### 1. Supabase Local Development

**Issue**: Cannot connect to local Supabase instance.

**Solution Pattern**:
```bash
# Check if Supabase is running
supabase status

# If not, start it
supabase start

# If you need to reset the database
supabase db reset
```

### 2. Styled Components TypeScript Integration

**Issue**: TypeScript errors with styled-components theme.

**Solution Pattern**:
Create a declaration file to extend the DefaultTheme interface from styled-components with your custom theme properties.

### 3. Testing Browser API Mocks

**Issue**: Tests fail with DOM-related errors.

**Solution Pattern**:
Include browser API mocks in your test setup file for commonly used browser APIs that may not be available in the test environment.

## First Steps

To begin development on the Bump application:

1. Clone the repository or create a new project using the Quick Start steps
2. Set up environment variables using the templates above
3. Review the detailed implementation guide for Milestone 1 in `.cursorguides/01-initialize-project.md`
4. Follow the step-by-step instructions to set up the initial project structure
5. Create the `.cursortrack.md` file to log your progress

## Next Steps

After completing the basic setup, proceed to the detailed implementation guide for the first milestone: Initialize Project. This will walk you through setting up the project structure, routing, theme system, and initial testing.

For detailed implementation steps, see [01-initialize-project.md](.cursorguides/01-initialize-project.md). 