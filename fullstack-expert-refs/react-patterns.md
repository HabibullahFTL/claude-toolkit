# React Patterns — Full Examples

<!--
  WHEN TO READ THIS FILE:
  - Building a form component for the first time in a project (to get the full ShadCN + RHF + Zod wiring).
  - User asks for a complete date formatting or form example.
  - Scenario A bootstrap: confirming the correct import paths and component structure.
  DO NOT read for simple tasks — the rules in the main prompt are sufficient for experienced use.
-->

---

## React Hook Form + Zod + ShadCN — Full Form Example

Install: `npm install react-hook-form @hookform/resolvers`

```tsx
'use client';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';

// Define schema next to the form file (or in _utils/) — never inline large schemas in the component body
const formSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email address'),
});

type FormValues = z.infer<typeof formSchema>;

export function ExampleForm() {
  const form = useForm<FormValues>({
    resolver: zodResolver(formSchema),
    defaultValues: { name: '', email: '' }, // always provide defaultValues to avoid uncontrolled→controlled warnings
  });

  const onSubmit = async (values: FormValues) => {
    // call mutation/action here — e.g. mutate(values) or call(values)
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input type="email" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        {/* Disable submit button while submitting to prevent double-submit */}
        <Button type="submit" disabled={form.formState.isSubmitting}>
          Submit
        </Button>
      </form>
    </Form>
  );
}
```

**Rules:**
- Always use `<Form>` + `<FormField>` + `<FormMessage>` from ShadCN for consistent error display
- Put the Zod schema next to the form file (or in `_utils/`) — never inline large schemas in the component body
- `defaultValues` must be provided to avoid uncontrolled→controlled warnings
- Use `form.formState.isSubmitting` to disable the submit button during submission

---

## date-fns — Usage Examples

Install: `npm install date-fns`

```tsx
import { format, formatDistanceToNow, isAfter, parseISO } from 'date-fns';

// Format for display
format(new Date(timestamp), 'MMM d, yyyy');                        // → "Apr 5, 2026"
format(new Date(timestamp), 'h:mm a');                             // → "2:30 PM"
format(new Date(timestamp), 'PPP');                                // → "April 5th, 2026"

// Relative time
formatDistanceToNow(new Date(timestamp), { addSuffix: true });    // → "3 days ago"

// Convex _creationTime is Unix ms — wrap with new Date() before passing to date-fns
const date = new Date(doc._creationTime);

// Comparisons
isAfter(new Date(endDate), new Date(startDate));
```

**Rules:**
- Convex `_creationTime` and stored timestamps are Unix ms numbers — always wrap with `new Date(value)`
- Never store pre-formatted date strings in the database — store timestamps, format at render time
- For timezone-aware work, use `date-fns-tz` and always be explicit about the timezone
- Never use `new Date().toLocaleDateString()` or raw `Date` methods in display code
