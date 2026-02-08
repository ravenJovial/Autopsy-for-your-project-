# Autopsy-for-your-project-
Project Autopsy is an AI-powered post-mortem tool that brutally reviews project ideas, GitHub repos, and hackathon pitches. It identifies fatal flaws, risky assumptions, overengineering, and wasted effort then tells you exactly what to cut to ship a stronger MVP.
src/ai/genkit.ts 

import {genkit} from 'genkit';
import {googleAI} from '@genkit-ai/google-genai';
import { z } from 'zod';

export const ai = genkit({
  plugins: [googleAI()],
  model: 'googleai/gemini-2.5-flash',
});


export const ProjectAutopsyInputSchema = z.object({
  projectIdea: z
    .string()
    .describe('The project idea, hackathon pitch, or a detailed description.'),
  githubUrl: z.string().optional().describe('An optional link to the GitHub repository for the project.'),
});
export type ProjectAutopsyInput = z.infer<typeof ProjectAutopsyInputSchema>;

export const ProjectAutopsyOutputSchema = z.object({
  overallVerdict: z.string().describe("A 1-2 sentence, brutally honest verdict on the project's viability."),
  fatalFlaws: z
    .array(z.string())
    .describe(
      'A list of critical, project-killing flaws. Things that will make judges or investors say "no" instantly.'
    ),
  riskyAssumptions: z
    .array(z.string())
    .describe(
      'A list of unproven assumptions the project relies on. If these are wrong, the project fails.'
    ),
  mvpCutList: z
    .array(z.string())
    .describe(
      'A list of features or aspects to cut to get to a minimum viable product. Focus on what is overengineered or what judges will not care about.'
    ),
});
export type ProjectAutopsyOutput = z.infer<typeof ProjectAutopsyOutputSchema>;

src/components/autopsy-studio.tsx

'use client';

import { useState } from 'react';
import { AutopsyForm } from '@/components/autopsy-form';
import { AutopsyReport } from '@/components/autopsy-report';
import { AppHeader } from '@/components/app-header';
import type { ProjectAutopsyOutput } from '@/ai/genkit';

export function AutopsyStudio() {
  const [autopsyResult, setAutopsyResult] = useState<ProjectAutopsyOutput | null>(
    null
  );

  return (
    <div className="flex flex-col min-h-screen">
      <AppHeader />
      <main className="flex-grow container mx-auto px-4 md:px-6 py-8">
        <div className="max-w-3xl mx-auto space-y-8">
          <AutopsyForm onAutopsyComplete={setAutopsyResult} />

          {autopsyResult && (
            <section className="animate-in fade-in-0 duration-500">
              <AutopsyReport result={autopsyResult} />
            </section>
          )}
        </div>
      </main>
    </div>
  );
}

scr/components/autopsy-report.tsx

'use client';
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger,
} from '@/components/ui/accordion';
import { Badge } from '@/components/ui/badge';
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import {
  AlertTriangle,
  Scissors,
  Scale,
  ListChecks,
} from 'lucide-react';
import type { ProjectAutopsyOutput } from '@/ai/genkit';

type AutopsyReportProps = {
  result: ProjectAutopsyOutput;
};

const sectionConfig = {
  fatalFlaws: {
    title: 'Fatal Flaws',
    Icon: AlertTriangle,
    badgeVariant: 'destructive',
    badgeText: 'Fatal',
    description: 'Critical issues that could kill your project before it starts.',
  },
  riskyAssumptions: {
    title: 'Risky Assumptions',
    Icon: Scale,
    badgeVariant: 'secondary',
    badgeText: 'Risky',
    description: 'Core assumptions that your project relies on to succeed. If these are wrong, you are in trouble.',
  },
  mvpCutList: {
    title: 'MVP Cut List',
    Icon: Scissors,
    badgeVariant: 'outline',
    badgeText: 'Cut',
    description: 'Features to cut or simplify to reach a Minimum Viable Product faster.',
  },
};

type SectionKey = keyof typeof sectionConfig;

export function AutopsyReport({ result }: AutopsyReportProps) {
  const sections = Object.keys(sectionConfig).filter(key => result[key as keyof ProjectAutopsyOutput] && (result[key as keyof ProjectAutopsyOutput] as string[]).length > 0) as SectionKey[];

  return (
    <Card className="border-border/50">
      <CardHeader>
        <CardTitle>Autopsy Report</CardTitle>
        <CardDescription>
          {result.overallVerdict}
        </CardDescription>
      </CardHeader>
      <CardContent>
        <Accordion type="multiple" defaultValue={sections} className="w-full">
          {sections.map((key) => {
            const config = sectionConfig[key];
            const items = result[key as keyof ProjectAutopsyOutput] as string[];
            return (
              <AccordionItem value={key} key={key}>
                <AccordionTrigger className="text-lg font-semibold hover:no-underline">
                  <div className="flex items-center gap-3">
                    <config.Icon className="h-5 w-5 text-primary" />
                    {config.title}
                  </div>
                </AccordionTrigger>
                <AccordionContent className="pt-2">
                  <p className="text-sm text-muted-foreground mb-4">{config.description}</p>
                  <ul className="space-y-3">
                    {items.map((item, index) => (
                      <li key={index} className="flex items-start gap-3 p-3 bg-muted/30 rounded-md">
                        <ListChecks className="h-4 w-4 mt-1 shrink-0 text-foreground/70" />
                        <div>
                          <Badge variant={config.badgeVariant} className="mb-2">{config.badgeText}</Badge>
                          <p className="text-foreground/90">{item}</p>
                        </div>
                      </li>
                    ))}
                  </ul>
                </AccordionContent>
              </AccordionItem>
            )
          })}
        </Accordion>
      </CardContent>
    </Card>
  );
}

src/components/autopsy-form.tsx

'use client';

import { useState } from 'react';
import { AutopsyForm } from '@/components/autopsy-form';
import { AutopsyReport } from '@/components/autopsy-report';
import { AppHeader } from '@/components/app-header';
import type { ProjectAutopsyOutput } from '@/ai/genkit';

export function AutopsyStudio() {
  const [autopsyResult, setAutopsyResult] = useState<ProjectAutopsyOutput | null>(
    null
  );

  return (
    <div className="flex flex-col min-h-screen">
      <AppHeader />
      <main className="flex-grow container mx-auto px-4 md:px-6 py-8">
        <div className="max-w-3xl mx-auto space-y-8">
          <AutopsyForm onAutopsyComplete={setAutopsyResult} />

          {autopsyResult && (
            <section className="animate-in fade-in-0 duration-500">
              <AutopsyReport result={autopsyResult} />
            </section>
          )}
        </div>
      </main>
    </div>
  );
}


src/app/action.ts
use server';

import { projectAutopsy } from '@/ai/flows/project-autopsy';
import { z } from 'zod';
import type { ProjectAutopsyOutput } from '@/ai/genkit';

const autopsySchema = z.object({
  projectIdea: z
    .string()
    .min(10, 'Please describe your idea in at least 10 characters.'),
  githubUrl: z.string().url('Please enter a valid URL.').optional().or(z.literal('')),
});

type AutopsyState = {
  message: string;
  errors: {
    projectIdea?: string[];
    githubUrl?: string[];
  };
  result: ProjectAutopsyOutput | null;
};

export async function autopsyAction(
  prevState: AutopsyState,
  formData: FormData
): Promise<AutopsyState> {
  const validatedFields = autopsySchema.safeParse({
    projectIdea: formData.get('projectIdea'),
    githubUrl: formData.get('githubUrl'),
  });

  if (!validatedFields.success) {
    return {
      message: 'Validation failed.',
      errors: validatedFields.error.flatten().fieldErrors,
      result: null,
    };
  }

  try {
    const result = await projectAutopsy({
      projectIdea: validatedFields.data.projectIdea,
      githubUrl: validatedFields.data.githubUrl,
    });
    return { message: 'success', result, errors: {} };
  } catch (error) {
    console.error(error);
    return {
      message: 'AI analysis failed.',
      result: null,
      errors: {},
    };
  }
}

Src/ai/flows/project-autopsy.ts
'use server';

/**
 * @fileOverview An AI agent that performs a "post-mortem" on a project idea, identifying potential flaws and risks.
 *
 * - projectAutopsy - A function that analyzes a project idea.
 */

import {
  ai,
  ProjectAutopsyInputSchema,
  ProjectAutopsyOutputSchema,
} from '@/ai/genkit';
import type { ProjectAutopsyInput, ProjectAutopsyOutput } from '@/ai/genkit';

export async function projectAutopsy(
  input: ProjectAutopsyInput
): Promise<ProjectAutopsyOutput> {
  return projectAutopsyFlow(input);
}

const prompt = ai.definePrompt({
  name: 'projectAutopsyPrompt',
  input: { schema: ProjectAutopsyInputSchema },
  output: { schema: ProjectAutopsyOutputSchema },
  prompt: `You are a cynical, brutally honest senior engineer and hackathon judge. Your job is to perform a "Project Autopsy" on the following idea. Do not sugarcoat anything. Be direct, concise, and ruthless in your feedback. Your goal is to expose the harsh truths the developer needs to hear before they waste their time.

Here is the project information:
Project Idea: {{{projectIdea}}}
{{#if githubUrl}}
GitHub Repository: {{{githubUrl}}}
{{/if}}

Structure your output exactly as follows in JSON format, adhering to the output schema:
1.  **overallVerdict**: A 1-2 sentence, brutally honest verdict on the project's viability.
2.  **fatalFlaws**: A list of critical, project-killing flaws. What will fail? What makes it an immediate "no"?
3.  **riskyAssumptions**: A list of unproven assumptions the project relies on. If these are wrong, the project fails.
4.  **mvpCutList**: A list of features to cut for an MVP. What's overengineered? What will judges not care about? What is just "fluff"?`,
});

const projectAutopsyFlow = ai.defineFlow(
  {
    name: 'projectAutopsyFlow',
    inputSchema: ProjectAutopsyInputSchema,
    outputSchema: ProjectAutopsyOutputSchema,
  },
  async (input) => {
    const { output } = await prompt(input);
    return output!;
  }
);

src/ai/flows/refine-generated-prompts.ts

'use server';

/**
 * @fileOverview An AI agent that performs a "post-mortem" on a project idea, identifying potential flaws and risks.
 *
 * - projectAutopsy - A function that analyzes a project idea.
 */

import {
  ai,
  ProjectAutopsyInputSchema,
  ProjectAutopsyOutputSchema,
} from '@/ai/genkit';
import type { ProjectAutopsyInput, ProjectAutopsyOutput } from '@/ai/genkit';

export async function projectAutopsy(
  input: ProjectAutopsyInput
): Promise<ProjectAutopsyOutput> {
  return projectAutopsyFlow(input);
}

const prompt = ai.definePrompt({
  name: 'projectAutopsyPrompt',
  input: { schema: ProjectAutopsyInputSchema },
  output: { schema: ProjectAutopsyOutputSchema },
  prompt: `You are a cynical, brutally honest senior engineer and hackathon judge. Your job is to perform a "Project Autopsy" on the following idea. Do not sugarcoat anything. Be direct, concise, and ruthless in your feedback. Your goal is to expose the harsh truths the developer needs to hear before they waste their time.

Here is the project information:
Project Idea: {{{projectIdea}}}
{{#if githubUrl}}
GitHub Repository: {{{githubUrl}}}
{{/if}}

Structure your output exactly as follows in JSON format, adhering to the output schema:
1.  **overallVerdict**: A 1-2 sentence, brutally honest verdict on the project's viability.
2.  **fatalFlaws**: A list of critical, project-killing flaws. What will fail? What makes it an immediate "no"?
3.  **riskyAssumptions**: A list of unproven assumptions the project relies on. If these are wrong, the project fails.
4.  **mvpCutList**: A list of features to cut for an MVP. What's overengineered? What will judges not care about? What is just "fluff"?`,
});

const projectAutopsyFlow = ai.defineFlow(
  {
    name: 'projectAutopsyFlow',
    inputSchema: ProjectAutopsyInputSchema,
    outputSchema: ProjectAutopsyOutputSchema,
  },
  async (input) => {
    const { output } = await prompt(input);
    return output!;
  }
);

src/ai/dev.ts
import { config } from 'dotenv';
config();

import '@/ai/flows/project-autopsy.ts';

tailwind.config.ts
import type {Config} from 'tailwindcss';

export default {
  darkMode: ['class'],
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        code: ['monospace'],
      },
      colors: {
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        chart: {
          '1': 'hsl(var(--chart-1))',
          '2': 'hsl(var(--chart-2))',
          '3': 'hsl(var(--chart-3))',
          '4': 'hsl(var(--chart-4))',
          '5': 'hsl(var(--chart-5))',
        },
        sidebar: {
          DEFAULT: 'hsl(var(--sidebar-background))',
          foreground: 'hsl(var(--sidebar-foreground))',
          primary: 'hsl(var(--sidebar-primary))',
          'primary-foreground': 'hsl(var(--sidebar-primary-foreground))',
          accent: 'hsl(var(--sidebar-accent))',
          'accent-foreground': 'hsl(var(--sidebar-accent-foreground))',
          border: 'hsl(var(--sidebar-border))',
          ring: 'hsl(var(--sidebar-ring))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      keyframes: {
        'accordion-down': {
          from: {
            height: '0',
          },
          to: {
            height: 'var(--radix-accordion-content-height)',
          },
        },
        'accordion-up': {
          from: {
            height: 'var(--radix-accordion-content-height)',
          },
          to: {
            height: '0',
          },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
} satisfies Config;

src/components/icon.tsx
import type { SVGProps } from 'react';

export function Logo(props: SVGProps<SVGSVGElement>) {
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
      {...props}
    >
      <circle cx="12" cy="12" r="10" />
      <line x1="22" x2="18" y1="12" y2="12" />
      <line x1="6" x2="2" y1="12" y2="12" />
      <line x1="12" x2="12" y1="6" y2="2" />
      <line x1="12" x2="12" y1="22" y2="18" />
    </svg>
  );
}

src/components/app-header.tsx
import { Logo } from '@/components/icons';

export function AppHeader() {
  return (
    <header className="py-6 px-4 md:px-6 border-b border-border/50">
      <div className="container mx-auto flex items-center gap-3">
        <Logo className="h-7 w-7 text-primary" />
        <h1 className="text-2xl font-bold text-foreground">
          Project Autopsy
        </h1>
      </div>
    </header>
  );
}
src/app/page.tsx
import { AutopsyStudio } from '@/components/autopsy-studio';

export default function Home() {
  return <AutopsyStudio />;
}

src/app/layout.tsx
import type { Metadata } from 'next';
import './globals.css';
import { Toaster } from '@/components/ui/toaster';

export const metadata: Metadata = {
  title: 'Project Autopsy',
  description:
    'An AI-powered post-mortem for your code and ideas. Get brutally honest feedback.',
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en" className="dark">
      <head>
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link
          rel="preconnect"
          href="https://fonts.gstatic.com"
          crossOrigin="anonymous"
        />
        <link
          href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap"
          rel="stylesheet"
        />
      </head>
      <body className="font-sans antialiased bg-background text-foreground">
        {children}
        <Toaster />
      </body>
    </html>
  );
}

src/app/globals.css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: 'Inter', sans-serif;
}

@layer base {
  :root {
    --background: 240 10% 3.9%;
    --foreground: 0 0% 98%;
    --card: 240 10% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 240 10% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 0 72% 51%; /* A strong red */
    --primary-foreground: 0 0% 98%;
    --secondary: 240 3.7% 15.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 240 3.7% 15.9%;
    --muted-foreground: 0 0% 63.9%;
    --accent: 240 3.7% 15.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 63% 31%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 3.7% 15.9%;
    --input: 240 3.7% 15.9%;
    --ring: 0 72% 51%;
    --chart-1: 220 70% 50%;
    --chart-2: 160 60% 45%;
    --chart-3: 30 80% 55%;
    --chart-4: 280 65% 60%;
    --chart-5: 340 75% 55%;
    --radius: 0.5rem;
  }
  .dark {
    --background: 240 10% 3.9%;
    --foreground: 0 0% 98%;
    --card: 240 10% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 240 10% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 0 72% 51%;
    --primary-foreground: 0 0% 98%;
    --secondary: 240 3.7% 15.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 240 3.7% 15.9%;
    --muted-foreground: 0 0% 63.9%;
    --accent: 240 3.7% 15.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 63% 31%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 3.7% 15.9%;
    --input: 240 3.7% 15.9%;
    --ring: 0 72% 51%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
