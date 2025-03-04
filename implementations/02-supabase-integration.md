# Milestone 2: Supabase Integration

## Overview
This milestone focuses on integrating Supabase as the backend for the Bump application. You'll be setting up the database schema, implementing authentication flows with OTP, creating client utilities, and building the foundation for real-time data features.

## Requirements

### 1. Supabase Project Setup

**Objective**: Create and configure a Supabase project for the Bump application.

**Requirements**:
- Create a new Supabase project in the dashboard
- Configure local development environment with Supabase CLI
- Set up environment variables for local and production environments
- Test connection to both local and production Supabase instances

**Configuration Needs**:
- Environment variables for URLs and API keys
- Separate configurations for development and testing
- Connection fallback strategies

**Example Environment Structure**:
```
# .env.development
VITE_SUPABASE_URL=<local_or_prod_url>
VITE_SUPABASE_ANON_KEY=<anon_key>
```

### 2. Database Schema Implementation

**Objective**: Create the database tables and relationships according to the schema specifications.

**Requirements**:
- Implement all tables specified in the DB Schema section
- Set up proper foreign key relationships with CASCADE rules
- Configure RLS (Row Level Security) policies for data access control
- Create necessary indexes for performance optimization
- Implement enum types for status fields like `hope_to_bump` and `notify_for`

**Tables to Implement**:
- `profiles` - User profile information (extends auth.users)
- `auth_otp` - OTP codes for email verification
- `places` - Gathering locations information
- `user_places` - Links users to their preferred places
- `statuses` - User check-ins with activity information
- `meetups` - Logged encounters between users
- `friends` - Friend connections with intent data
- `settings` - User preferences and configurations
- `notifications` - In-app notification storage

**Example RLS Policy Pattern**:
```sql
-- Example pattern for RLS policy (implement for each table)
CREATE POLICY "Users can view their own profiles"
ON profiles FOR SELECT
USING (auth.uid() = id);
```

### 3. Supabase Client Setup

**Objective**: Create a client utility for interacting with Supabase throughout the application.

**Requirements**:
- Create a singleton Supabase client instance
- Implement strong TypeScript types for database tables
- Set up error handling and connection resilience
- Configure client with environment variables
- Handle JWT token refresh

**Client Features**:
- TypeScript-based client with strong typing
- Error handling wrappers
- Connection state monitoring
- Support for both REST and Realtime subscriptions

**Example Client Pattern**:
```typescript
// Example type definition pattern (not complete)
export type Tables = {
  profiles: {
    Row: {
      id: string;
      email: string;
      phone: string | null;
      username: string;
      created_at: string;
    };
    Insert: {
      // Define insert type
    };
    Update: {
      // Define update type
    };
  };
  // Define other tables...
};
```

### 4. Authentication Implementation

**Objective**: Implement the OTP-based email authentication flow.

**Requirements**:
- Create email-based OTP authentication flow
- Implement OTP generation and verification
- Build rate limiting to prevent abuse
- Connect authentication to the AuthContext
- Handle session persistence and restoration
- Implement logout functionality

**Auth Flow Components**:
- Email submission form
- OTP verification screen
- Profile creation/update process
- Session management
- Error handling and retry mechanisms

**Security Considerations**:
- OTP expiration after 5 minutes
- Maximum of 3 verification attempts
- Rate limiting on email submissions
- Secure storage of authentication tokens

### 5. Polling Fallback for Realtime

**Objective**: Implement a fallback mechanism for environments where WebSockets aren't available.

**Requirements**:
- Create a polling mechanism for local development
- Design a toggle between WebSockets and polling
- Ensure consistent data interface regardless of fetch method
- Optimize polling frequency to balance UX and performance
- Handle connection state and recovery

**Implementation Strategies**:
- Environment-based connection selection
- Configurable polling intervals
- Connection health monitoring
- Transparent API for components (same interface for both methods)

### 6. CRUD Operations

**Objective**: Create utility functions for common database operations.

**Requirements**:
- Implement typed CRUD functions for all tables
- Create helper functions for common query patterns
- Build data transformation utilities
- Implement proper error handling
- Ensure all database operations respect RLS policies

**Operation Categories**:
- User profile management
- Places creation and discovery
- Status updates and retrieval
- Friend connections and intent management
- Settings persistence
- Notification handling

**Example Operation Pattern**:
```typescript
// Example operation pattern (not complete)
async function getProfileById(id: string) {
  // Implementation should handle:
  // 1. Type safety
  // 2. Error handling
  // 3. Null checking
  // 4. Return appropriate result
}
```

### 7. Testing Infrastructure

**Objective**: Set up testing utilities for Supabase integration.

**Requirements**:
- Create mock implementations of Supabase client
- Build test helpers for common Supabase operations
- Implement isolated test environments
- Configure test data fixtures
- Set up strategies for testing real-time features

**Testing Approaches**:
- Unit testing of client utilities
- Integration testing of authentication flows
- Mock-based testing of Supabase method chains
- Test environment with isolated database

## Verification Checklist

Before considering this milestone complete, ensure:

- [ ] All database tables are created and properly related
- [ ] RLS policies are applied to all tables
- [ ] Authentication flow works end-to-end
- [ ] Supabase client is properly configured with types
- [ ] Polling fallback works in local development
- [ ] CRUD operations for all tables are implemented and tested
- [ ] All tests related to Supabase integration pass
- [ ] The application can properly connect to both local and production Supabase
- [ ] Error handling is implemented for all Supabase operations
- [ ] Documentation is updated with Supabase integration details

## Next Steps

After completing this milestone, proceed to Milestone 3: Home & Check-In to implement the home page features and check-in functionality. 