# Hook Template

**Last Updated**: March 19, 2024  
**Related Guides**: [Testing Strategy](../concepts/testing-strategy.md)  
**Prerequisite Knowledge**: React Hooks, TypeScript, Supabase (if applicable)

## Overview

This template provides a standardized structure for creating custom React hooks in the Bump application. Following this template ensures hooks are consistent, well-typed, and properly tested.

## Hook Structure

### Basic Hook Template

```tsx
import { useState, useEffect, useCallback } from 'react';

// Define hook parameters and return type
export interface UseCounterOptions {
  /** Minimum value the counter can reach */
  minValue?: number;
  /** Maximum value the counter can reach */
  maxValue?: number;
  /** Step value for increment/decrement */
  step?: number;
}

export interface UseCounterReturn {
  /** Current count value */
  count: number;
  /** Increment the counter by step */
  increment: () => void;
  /** Decrement the counter by step */
  decrement: () => void;
  /** Reset the counter to initial value */
  reset: () => void;
  /** Set the counter to a specific value */
  setCount: (value: number) => void;
  /** Whether the counter is at its minimum value */
  isAtMin: boolean;
  /** Whether the counter is at its maximum value */
  isAtMax: boolean;
}

/**
 * Hook for managing a counter with min/max boundaries
 * 
 * @param initialValue - Starting value for the counter (default: 0)
 * @param options - Configuration options
 * @returns Counter state and methods
 * 
 * @example
 * ```tsx
 * const { count, increment, decrement, reset } = useCounter(10, { minValue: 0, maxValue: 20 });
 * ```
 */
export function useCounter(
  initialValue = 0,
  options: UseCounterOptions = {}
): UseCounterReturn {
  // Destructure options with defaults
  const { minValue = -Infinity, maxValue = Infinity, step = 1 } = options;
  
  // Ensure initial value respects boundaries
  const boundedInitialValue = Math.max(
    minValue,
    Math.min(maxValue, initialValue)
  );
  
  // State
  const [count, setCount] = useState<number>(boundedInitialValue);
  
  // Derived state
  const isAtMin = count <= minValue;
  const isAtMax = count >= maxValue;
  
  // Methods
  const increment = useCallback(() => {
    setCount(currentCount => {
      const newValue = currentCount + step;
      return newValue <= maxValue ? newValue : currentCount;
    });
  }, [maxValue, step]);
  
  const decrement = useCallback(() => {
    setCount(currentCount => {
      const newValue = currentCount - step;
      return newValue >= minValue ? newValue : currentCount;
    });
  }, [minValue, step]);
  
  const reset = useCallback(() => {
    setCount(boundedInitialValue);
  }, [boundedInitialValue]);
  
  // Bounded setter
  const setBoundedCount = useCallback((value: number) => {
    const boundedValue = Math.max(minValue, Math.min(maxValue, value));
    setCount(boundedValue);
  }, [minValue, maxValue]);
  
  // Return hook API
  return {
    count,
    increment,
    decrement,
    reset,
    setCount: setBoundedCount,
    isAtMin,
    isAtMax
  };
}

// Default export
export default useCounter;
```

### Data Fetching Hook Template

```tsx
import { useState, useEffect, useCallback } from 'react';
import { supabase } from '../utils/supabaseClient';

// Define hook parameters and return type
export interface UseProfileOptions {
  /** Whether to fetch immediately on mount */
  fetchOnMount?: boolean;
  /** Polling interval in milliseconds (0 to disable) */
  pollingInterval?: number;
}

export interface UseProfileReturn {
  /** Profile data */
  profile: Profile | null;
  /** Whether data is currently loading */
  isLoading: boolean;
  /** Error message if fetch failed */
  error: string | null;
  /** Manually fetch profile data */
  fetchProfile: () => Promise<void>;
  /** Update profile data */
  updateProfile: (data: Partial<Profile>) => Promise<void>;
}

/**
 * Hook for fetching and managing profile data
 * 
 * @param profileId - ID of the profile to fetch
 * @param options - Configuration options
 * @returns Profile data, loading state, and methods
 * 
 * @example
 * ```tsx
 * const { profile, isLoading, error, updateProfile } = useProfile('123');
 * ```
 */
export function useProfile(
  profileId: string,
  options: UseProfileOptions = {}
): UseProfileReturn {
  // Destructure options with defaults
  const { fetchOnMount = true, pollingInterval = 0 } = options;
  
  // State
  const [profile, setProfile] = useState<Profile | null>(null);
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);
  
  // Fetch profile data
  const fetchProfile = useCallback(async () => {
    if (!profileId) return;
    
    setIsLoading(true);
    setError(null);
    
    try {
      const { data, error } = await supabase
        .from('profiles')
        .select('*')
        .eq('id', profileId)
        .single();
      
      if (error) throw new Error(error.message);
      if (!data) throw new Error('Profile not found');
      
      setProfile(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An unknown error occurred');
      setProfile(null);
    } finally {
      setIsLoading(false);
    }
  }, [profileId]);
  
  // Update profile data
  const updateProfile = useCallback(async (data: Partial<Profile>) => {
    if (!profileId) return;
    
    setIsLoading(true);
    setError(null);
    
    try {
      // Validate data
      if (data.username === '') {
        throw new Error('Username cannot be empty');
      }
      
      const { data: updatedData, error } = await supabase
        .from('profiles')
        .update(data)
        .eq('id', profileId)
        .select()
        .single();
      
      if (error) throw new Error(error.message);
      if (!updatedData) throw new Error('Failed to update profile');
      
      setProfile(updatedData);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An unknown error occurred');
      throw err; // Re-throw for component handling
    } finally {
      setIsLoading(false);
    }
  }, [profileId]);
  
  // Fetch on mount
  useEffect(() => {
    if (fetchOnMount) {
      fetchProfile();
    }
  }, [fetchOnMount, fetchProfile]);
  
  // Setup polling if enabled
  useEffect(() => {
    if (pollingInterval <= 0) return;
    
    const intervalId = setInterval(fetchProfile, pollingInterval);
    
    return () => {
      clearInterval(intervalId);
    };
  }, [fetchProfile, pollingInterval]);
  
  // Return hook API
  return {
    profile,
    isLoading,
    error,
    fetchProfile,
    updateProfile
  };
}

// Default export
export default useProfile;
```

## Hook Test Template

```tsx
import { renderHook, act } from '@testing-library/react';
import { vi } from 'vitest';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });

  it('initializes with provided value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });

  it('increments the counter', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('respects maximum value', () => {
    const { result } = renderHook(() => useCounter(9, { maxValue: 10 }));
    
    act(() => {
      result.current.increment();
      result.current.increment(); // This should not take effect
    });
    
    expect(result.current.count).toBe(10);
    expect(result.current.isAtMax).toBe(true);
  });

  it('handles custom step value', () => {
    const { result } = renderHook(() => useCounter(0, { step: 5 }));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(5);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(5);
  });
});
```

## Best Practices

1. **Hook Design**
   - Use clear, descriptive names that start with "use"
   - Accept configuration options as a single object for flexibility
   - Return an object with named properties for clarity
   - Use TypeScript interfaces to define parameters and return types
   - Add JSDoc comments with examples

2. **State Management**
   - Use appropriate state initialization
   - Handle loading, error, and data states
   - Provide methods to update state
   - Use derived state for computed values

3. **Performance Optimization**
   - Memoize callbacks with useCallback
   - Memoize complex calculations with useMemo
   - Use dependency arrays correctly
   - Avoid unnecessary re-renders

4. **Side Effects**
   - Clean up side effects properly
   - Use appropriate dependency arrays
   - Handle async operations safely
   - Implement proper error handling

5. **Testing**
   - Test initialization with different parameters
   - Test all state changes and methods
   - Test side effects and cleanup
   - Test error handling

## Hook Checklist

Before considering a hook complete, ensure:

- [ ] Hook name starts with "use" and clearly describes its purpose
- [ ] Parameters and return values are properly typed
- [ ] JSDoc comments explain usage with examples
- [ ] Side effects are properly cleaned up
- [ ] Error states are handled appropriately
- [ ] Performance optimizations are applied where needed
- [ ] Tests cover initialization, state changes, and side effects

## Common Hook Patterns

### 1. Form Field Hook

```tsx
export function useField(initialValue = '') {
  const [value, setValue] = useState(initialValue);
  const [touched, setTouched] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const onChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  }, []);
  
  const onBlur = useCallback(() => {
    setTouched(true);
  }, []);
  
  const validate = useCallback((validator: (value: string) => string | null) => {
    const validationError = validator(value);
    setError(validationError);
    return !validationError;
  }, [value]);
  
  const reset = useCallback(() => {
    setValue(initialValue);
    setTouched(false);
    setError(null);
  }, [initialValue]);
  
  return {
    value,
    setValue,
    touched,
    error,
    onChange,
    onBlur,
    validate,
    reset,
    props: {
      value,
      onChange,
      onBlur
    }
  };
}
```

### 2. Local Storage Hook

```tsx
export function useLocalStorage<T>(key: string, initialValue: T) {
  // State to store our value
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      // Get from local storage by key
      const item = window.localStorage.getItem(key);
      // Parse stored json or if none return initialValue
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      // If error also return initialValue
      console.error(error);
      return initialValue;
    }
  });
  
  // Return a wrapped version of useState's setter function that
  // persists the new value to localStorage.
  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      // Allow value to be a function so we have same API as useState
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;
      // Save state
      setStoredValue(valueToStore);
      // Save to local storage
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      // A more advanced implementation would handle the error case
      console.error(error);
    }
  }, [key, storedValue]);
  
  // Subscribe to changes in other tabs/windows
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === key && e.newValue) {
        setStoredValue(JSON.parse(e.newValue));
      }
    };
    
    window.addEventListener('storage', handleStorageChange);
    
    return () => {
      window.removeEventListener('storage', handleStorageChange);
    };
  }, [key]);
  
  return [storedValue, setValue] as const;
}
```

### 3. Debounce Hook

```tsx
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    // Update debounced value after delay
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    // Cancel the timeout if value changes (also on delay change or unmount)
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}
```

## Related Patterns

- [Component Template](./component-template.md) - For creating components
- [Test Template](./test-template.md) - For writing comprehensive tests 