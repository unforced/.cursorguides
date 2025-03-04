# Utility Functions Template

**Last Updated**: March 19, 2024  
**Related Guides**: [Testing Strategy](../concepts/testing-strategy.md)  
**Prerequisite Knowledge**: TypeScript, JavaScript, Testing

## Overview

This template provides a standardized structure for creating utility functions in the Bump application. Following this template ensures utilities are consistent, well-typed, properly tested, and maintainable.

## Utility Function Structure

### Basic Utility Function Template

```ts
/**
 * Formats a date into a human-readable string
 * 
 * @param date - The date to format
 * @param format - The format to use (default: 'short')
 * @returns Formatted date string
 * 
 * @example
 * ```ts
 * // Returns "Mar 19, 2024"
 * formatDate(new Date(2024, 2, 19), 'short');
 * 
 * // Returns "Tuesday, March 19, 2024"
 * formatDate(new Date(2024, 2, 19), 'long');
 * 
 * // Returns "03/19/2024"
 * formatDate(new Date(2024, 2, 19), 'numeric');
 * ```
 */
export function formatDate(
  date: Date | string | number,
  format: 'short' | 'long' | 'numeric' | 'relative' = 'short'
): string {
  // Convert to Date object if string or number
  const dateObj = date instanceof Date ? date : new Date(date);
  
  // Validate date
  if (isNaN(dateObj.getTime())) {
    throw new Error('Invalid date provided');
  }
  
  // Format based on requested format
  switch (format) {
    case 'short':
      return dateObj.toLocaleDateString('en-US', {
        month: 'short',
        day: 'numeric',
        year: 'numeric'
      });
    
    case 'long':
      return dateObj.toLocaleDateString('en-US', {
        weekday: 'long',
        month: 'long',
        day: 'numeric',
        year: 'numeric'
      });
    
    case 'numeric':
      return dateObj.toLocaleDateString('en-US', {
        month: '2-digit',
        day: '2-digit',
        year: 'numeric'
      });
    
    case 'relative': {
      const now = new Date();
      const diffMs = now.getTime() - dateObj.getTime();
      const diffSec = Math.floor(diffMs / 1000);
      const diffMin = Math.floor(diffSec / 60);
      const diffHour = Math.floor(diffMin / 60);
      const diffDay = Math.floor(diffHour / 24);
      
      if (diffSec < 60) return 'just now';
      if (diffMin < 60) return `${diffMin}m ago`;
      if (diffHour < 24) return `${diffHour}h ago`;
      if (diffDay < 7) return `${diffDay}d ago`;
      
      return dateObj.toLocaleDateString('en-US', {
        month: 'short',
        day: 'numeric'
      });
    }
    
    default:
      return dateObj.toLocaleDateString();
  }
}

// Default export
export default formatDate;
```

### Validation Utility Function Template

```ts
/**
 * Validates an email address
 * 
 * @param email - The email address to validate
 * @returns True if valid, false otherwise
 * 
 * @example
 * ```ts
 * // Returns true
 * isValidEmail('user@example.com');
 * 
 * // Returns false
 * isValidEmail('invalid-email');
 * ```
 */
export function isValidEmail(email: string): boolean {
  if (!email || typeof email !== 'string') return false;
  
  // RFC 5322 compliant regex
  const emailRegex = /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;
  
  return emailRegex.test(email);
}

/**
 * Validates a phone number
 * 
 * @param phone - The phone number to validate
 * @returns True if valid, false otherwise
 * 
 * @example
 * ```ts
 * // Returns true
 * isValidPhone('(123) 456-7890');
 * isValidPhone('123-456-7890');
 * isValidPhone('1234567890');
 * 
 * // Returns false
 * isValidPhone('123-456');
 * ```
 */
export function isValidPhone(phone: string): boolean {
  if (!phone || typeof phone !== 'string') return false;
  
  // Remove all non-numeric characters
  const digits = phone.replace(/\D/g, '');
  
  // Check if we have 10-11 digits (US/Canada format)
  return digits.length >= 10 && digits.length <= 11;
}

/**
 * Formats a phone number to a consistent format
 * 
 * @param phone - The phone number to format
 * @returns Formatted phone number or empty string if invalid
 * 
 * @example
 * ```ts
 * // Returns "(123) 456-7890"
 * formatPhone('1234567890');
 * formatPhone('123-456-7890');
 * ```
 */
export function formatPhone(phone: string): string {
  if (!isValidPhone(phone)) return '';
  
  // Remove all non-numeric characters
  const digits = phone.replace(/\D/g, '');
  
  // Format as (XXX) XXX-XXXX
  const last10 = digits.slice(-10);
  const areaCode = last10.slice(0, 3);
  const middle = last10.slice(3, 6);
  const last = last10.slice(6);
  
  return `(${areaCode}) ${middle}-${last}`;
}
```

### Data Transformation Utility Function Template

```ts
/**
 * Groups an array of objects by a specified key
 * 
 * @param array - The array to group
 * @param key - The key to group by
 * @returns Object with groups as keys and arrays as values
 * 
 * @example
 * ```ts
 * const people = [
 *   { id: 1, city: 'Boulder' },
 *   { id: 2, city: 'Denver' },
 *   { id: 3, city: 'Boulder' }
 * ];
 * 
 * // Returns { Boulder: [{id:1,...}, {id:3,...}], Denver: [{id:2,...}] }
 * groupBy(people, 'city');
 * ```
 */
export function groupBy<T extends Record<string, any>, K extends keyof T>(
  array: T[],
  key: K
): Record<string, T[]> {
  return array.reduce((result, item) => {
    const groupKey = String(item[key]);
    
    if (!result[groupKey]) {
      result[groupKey] = [];
    }
    
    result[groupKey].push(item);
    return result;
  }, {} as Record<string, T[]>);
}

/**
 * Sorts an array of objects by a specified key
 * 
 * @param array - The array to sort
 * @param key - The key to sort by
 * @param direction - Sort direction (default: 'asc')
 * @returns Sorted array
 * 
 * @example
 * ```ts
 * const people = [
 *   { name: 'Charlie', age: 30 },
 *   { name: 'Alice', age: 25 },
 *   { name: 'Bob', age: 35 }
 * ];
 * 
 * // Returns sorted by name ascending
 * sortBy(people, 'name');
 * 
 * // Returns sorted by age descending
 * sortBy(people, 'age', 'desc');
 * ```
 */
export function sortBy<T extends Record<string, any>, K extends keyof T>(
  array: T[],
  key: K,
  direction: 'asc' | 'desc' = 'asc'
): T[] {
  return [...array].sort((a, b) => {
    if (a[key] < b[key]) return direction === 'asc' ? -1 : 1;
    if (a[key] > b[key]) return direction === 'asc' ? 1 : -1;
    return 0;
  });
}

/**
 * Filters an array of objects by a search term across multiple keys
 * 
 * @param array - The array to filter
 * @param searchTerm - The search term
 * @param keys - The keys to search in
 * @returns Filtered array
 * 
 * @example
 * ```ts
 * const people = [
 *   { name: 'Alice', bio: 'Engineer' },
 *   { name: 'Bob', bio: 'Designer' },
 *   { name: 'Charlie', bio: 'Product Manager' }
 * ];
 * 
 * // Returns [{ name: 'Alice', bio: 'Engineer' }]
 * searchObjects(people, 'eng', ['name', 'bio']);
 * ```
 */
export function searchObjects<T extends Record<string, any>>(
  array: T[],
  searchTerm: string,
  keys: (keyof T)[]
): T[] {
  if (!searchTerm) return array;
  
  const term = searchTerm.toLowerCase();
  
  return array.filter(item => {
    return keys.some(key => {
      const value = item[key];
      if (value == null) return false;
      return String(value).toLowerCase().includes(term);
    });
  });
}
```

## Utility Function Test Template

```ts
import { describe, it, expect } from 'vitest';
import { formatDate, isValidEmail, isValidPhone, groupBy } from './utils';

describe('formatDate', () => {
  it('formats date in short format', () => {
    const date = new Date(2024, 2, 19); // March 19, 2024
    expect(formatDate(date, 'short')).toBe('Mar 19, 2024');
  });

  it('formats date in long format', () => {
    const date = new Date(2024, 2, 19);
    expect(formatDate(date, 'long')).toBe('Tuesday, March 19, 2024');
  });

  it('formats date in numeric format', () => {
    const date = new Date(2024, 2, 19);
    expect(formatDate(date, 'numeric')).toBe('03/19/2024');
  });

  it('handles string dates', () => {
    expect(formatDate('2024-03-19', 'short')).toBe('Mar 19, 2024');
  });

  it('throws error for invalid dates', () => {
    expect(() => formatDate('invalid-date', 'short')).toThrow('Invalid date');
  });
});

describe('isValidEmail', () => {
  it('validates correct email addresses', () => {
    expect(isValidEmail('user@example.com')).toBe(true);
    expect(isValidEmail('user.name+tag@example.co.uk')).toBe(true);
  });

  it('rejects invalid email addresses', () => {
    expect(isValidEmail('user@')).toBe(false);
    expect(isValidEmail('user@example')).toBe(false);
    expect(isValidEmail('user example.com')).toBe(false);
    expect(isValidEmail('')).toBe(false);
  });
});

describe('isValidPhone', () => {
  it('validates correct phone numbers', () => {
    expect(isValidPhone('1234567890')).toBe(true);
    expect(isValidPhone('(123) 456-7890')).toBe(true);
    expect(isValidPhone('123-456-7890')).toBe(true);
  });

  it('rejects invalid phone numbers', () => {
    expect(isValidPhone('123-456')).toBe(false);
    expect(isValidPhone('abcdefghij')).toBe(false);
    expect(isValidPhone('')).toBe(false);
  });
});

describe('groupBy', () => {
  it('groups objects by key', () => {
    const items = [
      { id: 1, category: 'A' },
      { id: 2, category: 'B' },
      { id: 3, category: 'A' },
      { id: 4, category: 'C' }
    ];
    
    const result = groupBy(items, 'category');
    
    expect(result).toEqual({
      A: [{ id: 1, category: 'A' }, { id: 3, category: 'A' }],
      B: [{ id: 2, category: 'B' }],
      C: [{ id: 4, category: 'C' }]
    });
  });

  it('handles empty arrays', () => {
    expect(groupBy([], 'id')).toEqual({});
  });
});
```

## Best Practices

1. **Function Design**
   - Use clear, descriptive names that indicate purpose
   - Keep functions small and focused on a single task
   - Use TypeScript for type safety and documentation
   - Add JSDoc comments with examples
   - Use pure functions where possible (no side effects)

2. **Error Handling**
   - Validate inputs at the beginning of functions
   - Throw descriptive error messages for invalid inputs
   - Handle edge cases explicitly
   - Return sensible defaults for error cases when appropriate

3. **Performance Optimization**
   - Minimize unnecessary iterations and object creations
   - Use appropriate data structures for the task
   - Consider memoization for expensive calculations
   - Be mindful of time and space complexity

4. **Testing**
   - Test normal cases with expected inputs
   - Test edge cases and boundary conditions
   - Test error handling and invalid inputs
   - Use parameterized tests for multiple similar cases

5. **Organization**
   - Group related functions in the same file or module
   - Export named functions for better tree-shaking
   - Consider categorizing utilities (date, string, validation, etc.)
   - Document dependencies between utility functions

## Utility Function Checklist

Before considering a utility function complete, ensure:

- [ ] Function name clearly describes its purpose
- [ ] Parameters and return values are properly typed
- [ ] JSDoc comments explain usage with examples
- [ ] Input validation is implemented
- [ ] Edge cases are handled
- [ ] Tests cover normal cases, edge cases, and errors
- [ ] Function is pure when possible (no side effects)
- [ ] Performance considerations are addressed

## Common Utility Patterns

### 1. String Manipulation

```ts
/**
 * Truncates a string to a maximum length and adds ellipsis
 */
export function truncate(str: string, maxLength: number): string {
  if (!str) return '';
  if (str.length <= maxLength) return str;
  return str.slice(0, maxLength) + '...';
}

/**
 * Capitalizes the first letter of a string
 */
export function capitalize(str: string): string {
  if (!str) return '';
  return str.charAt(0).toUpperCase() + str.slice(1);
}

/**
 * Converts a string to title case
 */
export function titleCase(str: string): string {
  if (!str) return '';
  return str
    .toLowerCase()
    .split(' ')
    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');
}
```

### 2. Number Formatting

```ts
/**
 * Formats a number as currency
 */
export function formatCurrency(
  value: number,
  currency = 'USD',
  locale = 'en-US'
): string {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency
  }).format(value);
}

/**
 * Formats a number with thousands separators
 */
export function formatNumber(
  value: number,
  decimals = 0,
  locale = 'en-US'
): string {
  return new Intl.NumberFormat(locale, {
    minimumFractionDigits: decimals,
    maximumFractionDigits: decimals
  }).format(value);
}

/**
 * Clamps a number between min and max values
 */
export function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max);
}
```

### 3. Object Manipulation

```ts
/**
 * Creates a deep clone of an object
 */
export function deepClone<T>(obj: T): T {
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  if (Array.isArray(obj)) {
    return obj.map(deepClone) as unknown as T;
  }
  
  return Object.fromEntries(
    Object.entries(obj as Record<string, unknown>).map(
      ([key, value]) => [key, deepClone(value)]
    )
  ) as T;
}

/**
 * Picks specified properties from an object
 */
export function pick<T extends object, K extends keyof T>(
  obj: T,
  keys: K[]
): Pick<T, K> {
  return keys.reduce((result, key) => {
    if (key in obj) {
      result[key] = obj[key];
    }
    return result;
  }, {} as Pick<T, K>);
}

/**
 * Omits specified properties from an object
 */
export function omit<T extends object, K extends keyof T>(
  obj: T,
  keys: K[]
): Omit<T, K> {
  return Object.fromEntries(
    Object.entries(obj as Record<string, unknown>).filter(
      ([key]) => !keys.includes(key as K)
    )
  ) as Omit<T, K>;
}
```

## Related Patterns

- [Component Template](./component-template.md) - For creating components
- [Hook Template](./hook-template.md) - For creating custom hooks
- [Test Template](./test-template.md) - For writing comprehensive tests 