# Supabase Integration Template

**Last Updated**: March 19, 2024  
**Related Guides**: [Supabase Integration](../implementations/02-supabase-integration.md), [Supabase Testing](../troubleshooting/supabase-testing.md)  
**Prerequisite Knowledge**: Supabase, TypeScript, React Hooks

## Overview

This template provides standardized patterns for integrating Supabase in the Bump application. Following these patterns ensures consistent database operations, proper error handling, and effective real-time subscriptions.

## Client Setup

### Supabase Client Configuration

```ts
// src/utils/supabaseClient.ts

import { createClient } from '@supabase/supabase-js';
import type { Database } from '../types/supabase';

// Environment variables
const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

// Validate environment variables
if (!supabaseUrl || !supabaseAnonKey) {
  console.error(
    'Missing Supabase environment variables. Check your .env file.'
  );
}

// Create client with typed database schema
export const supabase = createClient<Database>(
  supabaseUrl,
  supabaseAnonKey,
  {
    auth: {
      persistSession: true,
      autoRefreshToken: true,
    },
    realtime: {
      params: {
        eventsPerSecond: 10,
      },
    },
  }
);

// Helper to check if we're in a development environment
export const isDevelopment = import.meta.env.DEV;

// Export typed helpers
export type Tables = Database['public']['Tables'];
export type Enums = Database['public']['Enums'];
```

### Database Types

```ts
// src/types/supabase.ts

export type Database = {
  public: {
    Tables: {
      profiles: {
        Row: {
          id: string;
          email: string;
          phone: string | null;
          username: string;
          created_at: string;
        };
        Insert: {
          id: string;
          email: string;
          phone?: string | null;
          username: string;
          created_at?: string;
        };
        Update: {
          id?: string;
          email?: string;
          phone?: string | null;
          username?: string;
          created_at?: string;
        };
      };
      places: {
        Row: {
          id: string;
          name: string;
          google_place_id: string;
          lat: number;
          lng: number;
        };
        Insert: {
          id?: string;
          name: string;
          google_place_id: string;
          lat: number;
          lng: number;
        };
        Update: {
          id?: string;
          name?: string;
          google_place_id?: string;
          lat?: number;
          lng?: number;
        };
      };
      // Add other tables as needed
    };
    Enums: {
      visibility_type: 'public' | 'friends' | 'private';
      hope_to_bump_type: 'off' | 'private' | 'shared';
      notify_type: 'all' | 'hope' | 'none';
    };
  };
};
```

## Authentication Patterns

### Email OTP Authentication

```tsx
// src/utils/auth.ts

import { supabase } from './supabaseClient';

/**
 * Send a one-time password to the user's email
 * 
 * @param email - User's email address
 * @returns Result object with success/error information
 */
export async function sendOTP(email: string) {
  try {
    // Validate email
    if (!email || !email.includes('@')) {
      return { success: false, error: 'Please enter a valid email address' };
    }
    
    // Send magic link
    const { error } = await supabase.auth.signInWithOtp({
      email,
      options: {
        emailRedirectTo: `${window.location.origin}/auth/callback`,
      },
    });
    
    if (error) throw error;
    
    return { success: true };
  } catch (error) {
    console.error('Error sending OTP:', error);
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Failed to send verification code' 
    };
  }
}

/**
 * Verify a one-time password
 * 
 * @param email - User's email address
 * @param otp - One-time password
 * @returns Result object with success/error/user information
 */
export async function verifyOTP(email: string, otp: string) {
  try {
    // Validate inputs
    if (!email || !otp) {
      return { success: false, error: 'Email and verification code are required' };
    }
    
    // Verify OTP
    const { data, error } = await supabase.auth.verifyOtp({
      email,
      token: otp,
      type: 'email',
    });
    
    if (error) throw error;
    
    return { success: true, user: data.user };
  } catch (error) {
    console.error('Error verifying OTP:', error);
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Failed to verify code' 
    };
  }
}

/**
 * Get the current authenticated user
 * 
 * @returns Current user or null if not authenticated
 */
export async function getCurrentUser() {
  try {
    const { data: { user }, error } = await supabase.auth.getUser();
    
    if (error) throw error;
    
    return user;
  } catch (error) {
    console.error('Error getting current user:', error);
    return null;
  }
}

/**
 * Sign out the current user
 * 
 * @returns Result object with success/error information
 */
export async function signOut() {
  try {
    const { error } = await supabase.auth.signOut();
    
    if (error) throw error;
    
    return { success: true };
  } catch (error) {
    console.error('Error signing out:', error);
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Failed to sign out' 
    };
  }
}
```

## Data Access Patterns

### Basic CRUD Operations

```tsx
// src/utils/profileService.ts

import { supabase } from './supabaseClient';
import type { Tables } from './supabaseClient';

type Profile = Tables['profiles']['Row'];
type ProfileUpdate = Tables['profiles']['Update'];

/**
 * Get a user's profile by ID
 * 
 * @param userId - User ID
 * @returns Profile data or null
 */
export async function getProfile(userId: string) {
  try {
    const { data, error } = await supabase
      .from('profiles')
      .select('*')
      .eq('id', userId)
      .single();
    
    if (error) throw error;
    
    return data;
  } catch (error) {
    console.error('Error fetching profile:', error);
    return null;
  }
}

/**
 * Create a new profile
 * 
 * @param profile - Profile data to insert
 * @returns Created profile or null
 */
export async function createProfile(profile: Tables['profiles']['Insert']) {
  try {
    // Validate required fields
    if (!profile.email || !profile.username) {
      throw new Error('Email and username are required');
    }
    
    const { data, error } = await supabase
      .from('profiles')
      .insert(profile)
      .select()
      .single();
    
    if (error) throw error;
    
    return data;
  } catch (error) {
    console.error('Error creating profile:', error);
    throw error; // Re-throw for component handling
  }
}

/**
 * Update a user's profile
 * 
 * @param userId - User ID
 * @param updates - Profile fields to update
 * @returns Updated profile or null
 */
export async function updateProfile(userId: string, updates: ProfileUpdate) {
  try {
    const { data, error } = await supabase
      .from('profiles')
      .update(updates)
      .eq('id', userId)
      .select()
      .single();
    
    if (error) throw error;
    
    return data;
  } catch (error) {
    console.error('Error updating profile:', error);
    throw error; // Re-throw for component handling
  }
}

/**
 * Delete a user's profile
 * 
 * @param userId - User ID
 * @returns Success status
 */
export async function deleteProfile(userId: string) {
  try {
    const { error } = await supabase
      .from('profiles')
      .delete()
      .eq('id', userId);
    
    if (error) throw error;
    
    return true;
  } catch (error) {
    console.error('Error deleting profile:', error);
    return false;
  }
}
```

### Filtering and Pagination

```tsx
// src/utils/placeService.ts

import { supabase } from './supabaseClient';
import type { Tables } from './supabaseClient';

type Place = Tables['places']['Row'];
type PlaceInsert = Tables['places']['Insert'];

/**
 * Get places with filtering and pagination
 * 
 * @param options - Query options
 * @returns Places and pagination info
 */
export async function getPlaces({
  page = 1,
  limit = 10,
  searchTerm = '',
  sortBy = 'name',
  sortDirection = 'asc'
}: {
  page?: number;
  limit?: number;
  searchTerm?: string;
  sortBy?: keyof Place;
  sortDirection?: 'asc' | 'desc';
} = {}) {
  try {
    // Calculate pagination
    const from = (page - 1) * limit;
    const to = from + limit - 1;
    
    // Start query
    let query = supabase
      .from('places')
      .select('*', { count: 'exact' });
    
    // Add search if provided
    if (searchTerm) {
      query = query.ilike('name', `%${searchTerm}%`);
    }
    
    // Add sorting and pagination
    const { data, error, count } = await query
      .order(sortBy, { ascending: sortDirection === 'asc' })
      .range(from, to);
    
    if (error) throw error;
    
    return {
      places: data,
      totalCount: count || 0,
      page,
      limit,
      totalPages: count ? Math.ceil(count / limit) : 0
    };
  } catch (error) {
    console.error('Error fetching places:', error);
    return {
      places: [],
      totalCount: 0,
      page,
      limit,
      totalPages: 0
    };
  }
}
```

### Joins and Complex Queries

```tsx
// src/utils/statusService.ts

import { supabase } from './supabaseClient';
import type { Tables } from './supabaseClient';

type Status = Tables['statuses']['Row'];
type StatusInsert = Tables['statuses']['Insert'];

/**
 * Get active statuses with user and place information
 * 
 * @returns Statuses with joined data
 */
export async function getActiveStatuses() {
  try {
    const { data, error } = await supabase
      .from('statuses')
      .select(`
        *,
        profiles:user_id (id, username),
        places:place_id (id, name, lat, lng)
      `)
      .eq('is_active', true)
      .order('timestamp', { ascending: false });
    
    if (error) throw error;
    
    return data;
  } catch (error) {
    console.error('Error fetching statuses:', error);
    return [];
  }
}

/**
 * Get statuses grouped by place
 * 
 * @returns Statuses grouped by place
 */
export async function getStatusesByPlace() {
  try {
    const { data, error } = await supabase
      .from('statuses')
      .select(`
        *,
        profiles:user_id (id, username),
        places:place_id (id, name, lat, lng)
      `)
      .eq('is_active', true)
      .order('timestamp', { ascending: false });
    
    if (error) throw error;
    
    // Group by place
    const groupedByPlace = data.reduce((acc, status) => {
      const placeId = status.place_id;
      
      if (!acc[placeId]) {
        acc[placeId] = {
          place: status.places,
          statuses: []
        };
      }
      
      acc[placeId].statuses.push({
        ...status,
        user: status.profiles
      });
      
      return acc;
    }, {} as Record<string, { place: any, statuses: any[] }>);
    
    return Object.values(groupedByPlace);
  } catch (error) {
    console.error('Error fetching statuses by place:', error);
    return [];
  }
}
```

## Real-time Subscriptions

### Basic Subscription Pattern

```tsx
// src/hooks/useRealtimeSubscription.ts

import { useState, useEffect } from 'react';
import { supabase, isDevelopment } from '../utils/supabaseClient';

/**
 * Hook for subscribing to real-time changes with polling fallback
 * 
 * @param tableName - Table to subscribe to
 * @param column - Column for filtering (optional)
 * @param value - Value for filtering (optional)
 * @param callback - Callback function for data changes
 * @param pollingInterval - Interval for polling in ms (default: 5000)
 * @returns Subscription status
 */
export function useRealtimeSubscription<T>(
  tableName: string,
  callback: (payload: { new: T, old: T }) => void,
  {
    column,
    value,
    event = '*',
    pollingInterval = 5000
  }: {
    column?: string;
    value?: any;
    event?: 'INSERT' | 'UPDATE' | 'DELETE' | '*';
    pollingInterval?: number;
  } = {}
) {
  const [isSubscribed, setIsSubscribed] = useState(false);
  
  // Setup real-time subscription
  useEffect(() => {
    let subscription: any;
    let pollingId: number;
    let lastFetchTime = 0;
    
    // Function to fetch latest data
    const fetchLatestData = async () => {
      try {
        let query = supabase.from(tableName).select('*');
        
        if (column && value !== undefined) {
          query = query.eq(column, value);
        }
        
        // Get data after the last fetch time
        if (lastFetchTime > 0) {
          query = query.gt('updated_at', new Date(lastFetchTime).toISOString());
        }
        
        const { data, error } = await query;
        
        if (error) throw error;
        
        // Update last fetch time
        lastFetchTime = Date.now();
        
        // Call callback for each new/updated record
        if (data && data.length > 0) {
          data.forEach(record => {
            callback({ new: record as T, old: {} as T });
          });
        }
      } catch (error) {
        console.error(`Error polling ${tableName}:`, error);
      }
    };
    
    // Setup subscription or polling based on environment
    const setupSubscription = async () => {
      // Use real-time subscription in production
      let channel = supabase
        .channel(`${tableName}_changes`)
        .on(
          'postgres_changes',
          {
            event,
            schema: 'public',
            table: tableName,
            ...(column && value !== undefined ? { filter: `${column}=eq.${value}` } : {})
          },
          payload => {
            callback(payload as any);
          }
        )
        .subscribe(status => {
          setIsSubscribed(status === 'SUBSCRIBED');
          
          // If subscription fails, fall back to polling
          if (status !== 'SUBSCRIBED' && isDevelopment) {
            console.warn(`Falling back to polling for ${tableName}`);
            pollingId = window.setInterval(fetchLatestData, pollingInterval);
          }
        });
      
      subscription = channel;
      
      // In development, use polling as a fallback
      if (isDevelopment) {
        // Initial fetch
        await fetchLatestData();
        
        // Setup polling
        pollingId = window.setInterval(fetchLatestData, pollingInterval);
      }
    };
    
    setupSubscription();
    
    // Cleanup
    return () => {
      if (subscription) {
        supabase.removeChannel(subscription);
      }
      
      if (pollingId) {
        clearInterval(pollingId);
      }
      
      setIsSubscribed(false);
    };
  }, [tableName, column, value, event, callback, pollingInterval]);
  
  return { isSubscribed };
}
```

### Using the Subscription Hook

```tsx
// Example component using real-time subscription

import { useState, useEffect } from 'react';
import { useRealtimeSubscription } from '../hooks/useRealtimeSubscription';
import { getActiveStatuses } from '../utils/statusService';
import type { Tables } from '../utils/supabaseClient';

type Status = Tables['statuses']['Row'] & {
  profiles: Tables['profiles']['Row'];
  places: Tables['places']['Row'];
};

export function StatusList() {
  const [statuses, setStatuses] = useState<Status[]>([]);
  const [loading, setLoading] = useState(true);
  
  // Initial data fetch
  useEffect(() => {
    const fetchStatuses = async () => {
      setLoading(true);
      const data = await getActiveStatuses();
      setStatuses(data as Status[]);
      setLoading(false);
    };
    
    fetchStatuses();
  }, []);
  
  // Subscribe to real-time updates
  const { isSubscribed } = useRealtimeSubscription<Status>(
    'statuses',
    (payload) => {
      if (payload.new && payload.new.id) {
        setStatuses(current => {
          // Check if this is an update to an existing status
          const existingIndex = current.findIndex(s => s.id === payload.new.id);
          
          if (existingIndex >= 0) {
            // Update existing status
            const updated = [...current];
            updated[existingIndex] = payload.new;
            return updated;
          } else {
            // Add new status
            return [payload.new, ...current];
          }
        });
      }
    },
    { 
      column: 'is_active', 
      value: true,
      pollingInterval: 3000 
    }
  );
  
  if (loading) {
    return <div>Loading statuses...</div>;
  }
  
  return (
    <div>
      <h2>Active Statuses {isSubscribed ? '(Live)' : ''}</h2>
      {statuses.length === 0 ? (
        <p>No active statuses</p>
      ) : (
        <ul>
          {statuses.map(status => (
            <li key={status.id}>
              {status.profiles.username} at {status.places.name}: {status.activity}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## Error Handling Patterns

### Consistent Error Handling

```tsx
// src/utils/errorHandling.ts

/**
 * Format Supabase error for display
 * 
 * @param error - Error object from Supabase
 * @returns User-friendly error message
 */
export function formatSupabaseError(error: any): string {
  if (!error) return 'An unknown error occurred';
  
  // Handle Supabase error object
  if (error.code) {
    switch (error.code) {
      case '23505': // Unique violation
        return 'This record already exists';
      case '23503': // Foreign key violation
        return 'This operation references a record that does not exist';
      case '42P01': // Undefined table
        return 'Database configuration error';
      case 'PGRST116': // Row not found
        return 'Record not found';
      case '22P02': // Invalid text representation
        return 'Invalid input format';
      default:
        return error.message || `Database error: ${error.code}`;
    }
  }
  
  // Handle auth errors
  if (error.name === 'AuthApiError') {
    switch (error.status) {
      case 400:
        return 'Invalid credentials';
      case 401:
        return 'You are not authorized to perform this action';
      case 404:
        return 'User not found';
      case 429:
        return 'Too many requests, please try again later';
      default:
        return error.message || 'Authentication error';
    }
  }
  
  // Handle network errors
  if (error.name === 'TypeError' && error.message.includes('fetch')) {
    return 'Network error. Please check your connection';
  }
  
  // Handle generic errors
  return error.message || 'An unexpected error occurred';
}

/**
 * Wrap a database operation with consistent error handling
 * 
 * @param operation - Async database operation
 * @param errorHandler - Custom error handler (optional)
 * @returns Result of the operation or error
 */
export async function withErrorHandling<T>(
  operation: () => Promise<T>,
  errorHandler?: (error: any) => void
): Promise<{ data: T | null; error: string | null }> {
  try {
    const data = await operation();
    return { data, error: null };
  } catch (error) {
    const errorMessage = formatSupabaseError(error);
    
    // Call custom error handler if provided
    if (errorHandler) {
      errorHandler(error);
    } else {
      console.error('Database operation failed:', errorMessage, error);
    }
    
    return { data: null, error: errorMessage };
  }
}
```

### Using Error Handling

```tsx
// Example of using error handling

import { withErrorHandling } from '../utils/errorHandling';
import { supabase } from '../utils/supabaseClient';

export async function createPlace(place) {
  return withErrorHandling(async () => {
    const { data, error } = await supabase
      .from('places')
      .insert(place)
      .select()
      .single();
    
    if (error) throw error;
    
    return data;
  });
}

// In a component
async function handleSubmit() {
  setLoading(true);
  
  const { data, error } = await createPlace({
    name: placeName,
    google_place_id: googlePlaceId,
    lat: location.lat,
    lng: location.lng
  });
  
  setLoading(false);
  
  if (error) {
    setError(error);
    return;
  }
  
  // Success
  setSuccess(true);
  navigate('/places');
}
```

## Best Practices

1. **Client Setup**
   - Use typed Supabase client with database schema
   - Store environment variables properly
   - Configure client options for your use case
   - Implement development vs. production detection

2. **Authentication**
   - Implement proper validation before auth calls
   - Handle errors consistently
   - Provide clear user feedback
   - Secure sensitive operations

3. **Data Access**
   - Use typed tables and operations
   - Implement consistent error handling
   - Structure complex queries for readability
   - Consider performance for large datasets

4. **Real-time Features**
   - Implement polling fallback for development
   - Handle subscription lifecycle properly
   - Update UI efficiently with new data
   - Clean up subscriptions on component unmount

5. **Error Handling**
   - Format errors for user display
   - Log detailed errors for debugging
   - Handle specific error codes appropriately
   - Provide recovery paths when possible

## Supabase Integration Checklist

Before considering a Supabase integration complete, ensure:

- [ ] Database schema is properly typed
- [ ] Authentication flow is secure and user-friendly
- [ ] CRUD operations are implemented with proper error handling
- [ ] Real-time subscriptions work with polling fallback
- [ ] Complex queries are optimized and readable
- [ ] Tests cover success and error cases
- [ ] Environment variables are documented

## Related Patterns

- [Hook Template](./hook-template.md) - For creating custom hooks
- [Test Template](./test-template.md) - For writing comprehensive tests
- [Utility Functions Template](./utility-functions-template.md) - For creating utility functions 