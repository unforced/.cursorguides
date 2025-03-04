# Test Template

**Last Updated**: March 19, 2024  
**Related Guides**: [Testing Strategy](../concepts/testing-strategy.md), [Supabase Testing](../troubleshooting/supabase-testing.md)  
**Prerequisite Knowledge**: Vitest, React Testing Library, TypeScript

## Overview

This template provides a standardized structure for writing tests in the Bump application. Following this template ensures comprehensive test coverage and consistent testing patterns across the codebase.

## Test Types

### 1. Component Tests

```tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ThemeProvider } from 'styled-components';
import { theme } from '../../styles/theme';
import { ProfileCard } from './ProfileCard';

// Mock data
const mockProfile = {
  id: '123',
  username: 'testuser',
  email: 'test@example.com',
  created_at: '2024-03-01T12:00:00Z'
};

// Helper function to render with theme
const renderWithTheme = (ui: React.ReactElement) => {
  return render(<ThemeProvider theme={theme}>{ui}</ThemeProvider>);
};

describe('ProfileCard', () => {
  // Setup and teardown
  beforeEach(() => {
    // Common setup code
    vi.clearAllMocks();
  });

  // Rendering tests
  it('renders profile information correctly', () => {
    renderWithTheme(<ProfileCard profile={mockProfile} />);
    
    // Check that key elements are rendered
    expect(screen.getByText('testuser')).toBeInTheDocument();
    expect(screen.getByText('test@example.com')).toBeInTheDocument();
    expect(screen.getByRole('img')).toHaveAttribute('alt', 'Profile for testuser');
  });

  // Interaction tests
  it('calls onEdit when edit button is clicked', async () => {
    const handleEdit = vi.fn();
    renderWithTheme(<ProfileCard profile={mockProfile} onEdit={handleEdit} />);
    
    // Use userEvent for better interaction simulation
    const user = userEvent.setup();
    await user.click(screen.getByRole('button', { name: /edit/i }));
    
    expect(handleEdit).toHaveBeenCalledTimes(1);
    expect(handleEdit).toHaveBeenCalledWith(mockProfile.id);
  });

  // State tests
  it('shows loading state when isLoading is true', () => {
    renderWithTheme(<ProfileCard profile={mockProfile} isLoading={true} />);
    
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
    expect(screen.queryByText('testuser')).not.toBeInTheDocument();
  });

  // Error state tests
  it('shows error message when error is provided', () => {
    renderWithTheme(<ProfileCard profile={mockProfile} error="Failed to load profile" />);
    
    expect(screen.getByText('Failed to load profile')).toBeInTheDocument();
  });

  // Conditional rendering tests
  it('conditionally renders edit button based on canEdit prop', () => {
    const { rerender } = renderWithTheme(
      <ProfileCard profile={mockProfile} canEdit={true} />
    );
    
    expect(screen.getByRole('button', { name: /edit/i })).toBeInTheDocument();
    
    rerender(
      <ThemeProvider theme={theme}>
        <ProfileCard profile={mockProfile} canEdit={false} />
      </ThemeProvider>
    );
    
    expect(screen.queryByRole('button', { name: /edit/i })).not.toBeInTheDocument();
  });

  // Accessibility tests
  it('has appropriate ARIA attributes', () => {
    renderWithTheme(<ProfileCard profile={mockProfile} />);
    
    const card = screen.getByRole('article');
    expect(card).toHaveAttribute('aria-labelledby', expect.any(String));
    
    const heading = screen.getByRole('heading');
    expect(heading.id).toBe(card.getAttribute('aria-labelledby'));
  });
});
```

### 2. Hook Tests

```tsx
import { renderHook, act } from '@testing-library/react';
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

  it('decrements the counter', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('resets the counter', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(5);
  });

  it('does not go below 0 when minValue is set', () => {
    const { result } = renderHook(() => useCounter(1, { minValue: 0 }));
    
    act(() => {
      result.current.decrement();
      result.current.decrement(); // This should not take effect
    });
    
    expect(result.current.count).toBe(0);
  });
});
```

### 3. Utility Function Tests

```tsx
import { formatDate, validateEmail, truncateText } from './utils';

describe('formatDate', () => {
  it('formats date in the expected format', () => {
    const date = new Date('2024-03-19T12:00:00Z');
    expect(formatDate(date)).toBe('March 19, 2024');
  });

  it('handles invalid date input', () => {
    expect(formatDate(null)).toBe('Invalid date');
    expect(formatDate(undefined)).toBe('Invalid date');
    expect(formatDate('not a date')).toBe('Invalid date');
  });
});

describe('validateEmail', () => {
  it('returns true for valid email addresses', () => {
    expect(validateEmail('test@example.com')).toBe(true);
    expect(validateEmail('user.name+tag@domain.co.uk')).toBe(true);
  });

  it('returns false for invalid email addresses', () => {
    expect(validateEmail('test')).toBe(false);
    expect(validateEmail('test@')).toBe(false);
    expect(validateEmail('@example.com')).toBe(false);
    expect(validateEmail('test@example')).toBe(false);
  });

  it('handles edge cases', () => {
    expect(validateEmail('')).toBe(false);
    expect(validateEmail(null)).toBe(false);
    expect(validateEmail(undefined)).toBe(false);
  });
});

describe('truncateText', () => {
  it('truncates text longer than the specified length', () => {
    expect(truncateText('This is a long text', 10)).toBe('This is a...');
  });

  it('does not truncate text shorter than the specified length', () => {
    expect(truncateText('Short', 10)).toBe('Short');
  });

  it('uses custom suffix when provided', () => {
    expect(truncateText('This is a long text', 10, '---')).toBe('This is a---');
  });
});
```

### 4. Supabase Integration Tests

```tsx
import { vi } from 'vitest';
import { getProfile, updateProfile } from './profileService';
import { createMockSupabase } from '../test/mocks/supabaseMock';

// Mock Supabase client
vi.mock('../utils/supabaseClient', () => {
  return {
    supabase: createMockSupabase({
      profiles: [
        { id: '123', username: 'testuser', email: 'test@example.com' }
      ]
    })
  };
});

describe('Profile Service', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('getProfile', () => {
    it('returns profile when found', async () => {
      const profile = await getProfile('123');
      
      expect(profile).toEqual({
        id: '123',
        username: 'testuser',
        email: 'test@example.com'
      });
    });

    it('handles not found error', async () => {
      // Override the mock for this specific test
      const { supabase } = await import('../utils/supabaseClient');
      vi.mocked(supabase.from).mockImplementationOnce(() => {
        return {
          select: vi.fn().mockReturnValue({
            eq: vi.fn().mockResolvedValue({ data: null, error: null })
          })
        };
      });
      
      await expect(getProfile('456')).rejects.toThrow('Profile not found');
    });

    it('handles database error', async () => {
      // Override the mock for this specific test
      const { supabase } = await import('../utils/supabaseClient');
      vi.mocked(supabase.from).mockImplementationOnce(() => {
        return {
          select: vi.fn().mockReturnValue({
            eq: vi.fn().mockResolvedValue({
              data: null,
              error: { message: 'Database error' }
            })
          })
        };
      });
      
      await expect(getProfile('123')).rejects.toThrow('Database error');
    });
  });

  describe('updateProfile', () => {
    it('updates profile successfully', async () => {
      const updatedProfile = {
        username: 'newusername',
        bio: 'New bio'
      };
      
      const result = await updateProfile('123', updatedProfile);
      
      expect(result).toEqual({
        id: '123',
        ...updatedProfile
      });
    });

    it('handles validation errors', async () => {
      await expect(updateProfile('123', { username: '' }))
        .rejects.toThrow('Username cannot be empty');
    });
  });
});
```

## Best Practices

1. **Test Organization**
   - Group tests logically with `describe` blocks
   - Use clear, descriptive test names that explain the expected behavior
   - Follow the Arrange-Act-Assert pattern within tests
   - Keep tests independent and isolated

2. **Component Testing**
   - Test rendering, interactions, and state changes
   - Use `screen` queries to find elements (prefer accessible queries)
   - Use `userEvent` over `fireEvent` for more realistic interactions
   - Test all component states (loading, error, empty, etc.)

3. **Hook Testing**
   - Use `renderHook` to test custom hooks
   - Wrap state changes in `act` to ensure proper updates
   - Test initialization, state changes, and side effects

4. **Utility Testing**
   - Test both valid and invalid inputs
   - Test edge cases and boundary conditions
   - Group related tests with descriptive `describe` blocks

5. **Mocking**
   - Mock external dependencies and services
   - Reset mocks between tests with `vi.clearAllMocks()`
   - Use specific mocks for specific tests when needed
   - For Supabase, use the patterns from the Supabase Testing guide

## Test Checklist

Before considering tests complete, ensure:

- [ ] All component props and states are tested
- [ ] User interactions are tested with `userEvent`
- [ ] Error states and edge cases are covered
- [ ] Accessibility concerns are tested
- [ ] Mocks are properly set up and cleared between tests
- [ ] Tests are independent and don't rely on each other
- [ ] Test coverage meets project standards (>80%)

## Common Testing Patterns

### Testing Asynchronous Code

```tsx
it('loads data asynchronously', async () => {
  // Arrange
  renderWithTheme(<DataComponent />);
  
  // Act - nothing explicit, component loads data on mount
  
  // Assert - wait for loading to complete
  expect(await screen.findByText('Loaded Data')).toBeInTheDocument();
});
```

### Testing Form Submission

```tsx
it('submits form with valid data', async () => {
  // Arrange
  const handleSubmit = vi.fn();
  renderWithTheme(<Form onSubmit={handleSubmit} />);
  const user = userEvent.setup();
  
  // Act
  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password123');
  await user.click(screen.getByRole('button', { name: /submit/i }));
  
  // Assert
  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123'
  });
});
```

### Testing Error Handling

```tsx
it('displays error message when API call fails', async () => {
  // Arrange - mock API to fail
  fetchMock.mockRejectedValueOnce(new Error('API Error'));
  
  // Act
  renderWithTheme(<DataComponent />);
  
  // Assert
  expect(await screen.findByText('Failed to load data: API Error')).toBeInTheDocument();
});
```

## Related Patterns

- [Component Template](./component-template.md) - For creating components
- [Hook Template](./hook-template.md) - For creating custom hooks 