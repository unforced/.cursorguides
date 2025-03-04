# Component Template

**Last Updated**: March 19, 2024  
**Related Guides**: [Testing Strategy](../concepts/testing-strategy.md)  
**Prerequisite Knowledge**: React, TypeScript, styled-components

## Overview

This template provides a standardized structure for creating React components in the Bump application. Following this template ensures consistency across components and makes them easier to test, maintain, and extend.

## Component Structure

### Basic Component Template

```tsx
import React from 'react';
import styled from 'styled-components';
import { ComponentProps } from '../../types/components';

// Define component props with TypeScript
export interface ButtonProps extends ComponentProps {
  /** Primary content of the button */
  children: React.ReactNode;
  /** Called when the button is clicked */
  onClick?: () => void;
  /** Visual variant of the button */
  variant?: 'primary' | 'secondary' | 'text';
  /** Whether the button is in a loading state */
  isLoading?: boolean;
  /** Whether the button is disabled */
  disabled?: boolean;
}

// Create styled components
const StyledButton = styled.button<{ variant: ButtonProps['variant'] }>`
  padding: ${({ theme }) => `${theme.space[2]} ${theme.space[3]}`};
  border-radius: ${({ theme }) => theme.radii.md};
  font-weight: ${({ theme }) => theme.fontWeights.bold};
  font-size: ${({ theme }) => theme.fontSizes.body};
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  
  /* Apply styles based on variant */
  ${({ theme, variant }) => {
    switch (variant) {
      case 'primary':
        return `
          background-color: ${theme.colors.primary};
          color: ${theme.colors.white};
          border: none;
          
          &:hover {
            background-color: ${theme.colors.primaryDark};
          }
        `;
      case 'secondary':
        return `
          background-color: transparent;
          color: ${theme.colors.primary};
          border: 1px solid ${theme.colors.primary};
          
          &:hover {
            background-color: ${theme.colors.primaryLight};
          }
        `;
      case 'text':
        return `
          background-color: transparent;
          color: ${theme.colors.primary};
          border: none;
          padding: ${theme.space[1]} ${theme.space[2]};
          
          &:hover {
            text-decoration: underline;
          }
        `;
      default:
        return '';
    }
  }}
  
  /* Disabled state */
  ${({ disabled, theme }) =>
    disabled &&
    `
    opacity: 0.5;
    cursor: not-allowed;
    &:hover {
      background-color: ${theme.colors.primary};
    }
  `}
`;

const LoadingSpinner = styled.div`
  /* Spinner styles */
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s ease-in-out infinite;
  margin-right: ${({ theme }) => theme.space[2]};
  
  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }
`;

/**
 * Button component for user interactions
 */
export const Button: React.FC<ButtonProps> = ({
  children,
  onClick,
  variant = 'primary',
  isLoading = false,
  disabled = false,
  ...props
}) => {
  return (
    <StyledButton
      variant={variant}
      onClick={onClick}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading && <LoadingSpinner data-testid="loading-spinner" />}
      {children}
    </StyledButton>
  );
};

// Default export
export default Button;
```

### Component Test Template

```tsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import { ThemeProvider } from 'styled-components';
import { theme } from '../../styles/theme';
import Button from './Button';

// Helper function to render with theme
const renderWithTheme = (ui: React.ReactElement) => {
  return render(<ThemeProvider theme={theme}>{ui}</ThemeProvider>);
};

describe('Button', () => {
  it('renders correctly with children', () => {
    renderWithTheme(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    renderWithTheme(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('shows loading spinner when isLoading is true', () => {
    renderWithTheme(<Button isLoading>Loading</Button>);
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });

  it('is disabled when disabled prop is true', () => {
    renderWithTheme(<Button disabled>Disabled</Button>);
    expect(screen.getByText('Disabled')).toBeDisabled();
  });

  it('is disabled when isLoading is true', () => {
    renderWithTheme(<Button isLoading>Loading</Button>);
    expect(screen.getByText('Loading')).toBeDisabled();
  });

  it('renders with different variants', () => {
    const { rerender } = renderWithTheme(<Button variant="primary">Primary</Button>);
    expect(screen.getByText('Primary')).toHaveStyle('background-color: #2E7D32');

    rerender(
      <ThemeProvider theme={theme}>
        <Button variant="secondary">Secondary</Button>
      </ThemeProvider>
    );
    expect(screen.getByText('Secondary')).toHaveStyle('background-color: transparent');

    rerender(
      <ThemeProvider theme={theme}>
        <Button variant="text">Text</Button>
      </ThemeProvider>
    );
    expect(screen.getByText('Text')).toHaveStyle('background-color: transparent');
  });
});
```

## Best Practices

1. **Props Interface**
   - Always define a TypeScript interface for component props
   - Use JSDoc comments to document each prop
   - Extend from `ComponentProps` for common props
   - Provide sensible defaults for optional props

2. **Styled Components**
   - Use the theme for all styling values (colors, spacing, etc.)
   - Create separate styled components for complex elements
   - Use props to control styling variations
   - Keep styled components in the same file unless they're shared

3. **Component Implementation**
   - Use functional components with React.FC type
   - Destructure props with defaults
   - Use data-testid for test-specific selectors
   - Handle loading and error states
   - Spread remaining props to the root element

4. **Testing**
   - Test rendering with different props
   - Test user interactions
   - Test all visual states (loading, disabled, variants)
   - Use renderWithTheme helper for consistent theme

## Component Checklist

Before considering a component complete, ensure:

- [ ] Props are properly typed with JSDoc comments
- [ ] Component handles all states (normal, loading, error, disabled)
- [ ] Styling uses theme values and is responsive
- [ ] Tests cover rendering, interactions, and all states
- [ ] Component is accessible (keyboard navigation, ARIA attributes)
- [ ] Performance considerations (memoization if needed)

## Example Usage

```tsx
import Button from '../components/Button';

const HomePage = () => {
  const [isLoading, setIsLoading] = useState(false);
  
  const handleClick = async () => {
    setIsLoading(true);
    try {
      await someAsyncOperation();
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <div>
      <Button 
        onClick={handleClick} 
        isLoading={isLoading}
        variant="primary"
      >
        Submit
      </Button>
      
      <Button variant="text">Cancel</Button>
    </div>
  );
};
```

## Related Patterns

- [Hook Template](./hook-template.md) - For creating custom hooks
- [Test Template](./test-template.md) - For writing comprehensive tests 