Logging is a critical aspect of modern application development, providing visibility into the behavior and performance of your apps—both on the frontend and backend. By capturing essential data about requests, errors, and system events, logging helps developers troubleshoot issues quickly and optimize user experiences.

At Infinum, where we leverage technologies like Next.js, Angular, NestJS and AWS, we recognize the importance of robust, efficient, and scalable logging. After evaluating several popular logging libraries, we standardized on Pino as our primary choice for Node.js applications. Below, we compare Pino with other leading solutions and detail why we chose it.

## Pino vs. other solutions

To make an informed decision, we compared three widely-used logging solutions: [Pino](https://getpino.io/#/), [Winston](https://github.com/winstonjs/winston), and [Graylog](https://graylog.org/). Each offers distinct advantages and trade-offs.

| Feature/Aspect          | Pino                                                                                                                          | Winston                                                                        | Graylog                                                                                        |
|-------------------------|-------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| Performance             | Optimized for speed with minimal overhead; ideal for real-time apps and SSR.                                                  | Slower due to a more feature-rich architecture.                                | Designed for centralized log aggregation, not Node.js performance.                             |
| JSON Logging            | Native JSON logging for structured data.                                                                                      | JSON is supported but requires more setup.                                     | Centralized logging solution; inherently JSON-based but requires standalone infrastructure.    |
| Integration with NestJS | Excellent with [nestjs-pino](https://github.com/iamolegga/nestjs-pino). Seamless middleware and dependency injection support. | Good with nestjs-winston, but heavier setup.                                   | Not a direct logger; requires a Graylog server or service.                                     |
| AWS Compatibility       | JSON-first approach simplifies usage with AWS services like CloudWatch, Lambda, and S3.                                       | Possible, but typically needs additional config.                               | Enterprise-level logging solution with its own setup, not directly tied to AWS.                |
| Ease of Use             | Simple, minimal boilerplate.                                                                                                  | Highly configurable but more complex to set up.                                | Requires installing and configuring a Graylog server, which can be overkill for many projects. |
| Development Experience  | Lightweight, with [pino-pretty](https://github.com/pinojs/pino-pretty) available for easy debugging.                          | Debugging is less streamlined out of the box.                                  | Focused on centralized logs; not meant for local debugging.                                    |
| Scalability             | Scales effortlessly in microservices or distributed systems.                                                                  | Scales reasonably but can add overhead.                                        | Centralized solution designed for large-scale log management, but introduces latency.          |
| Logging Destination     | Console, files, or integrations with ELK, AWS, etc.                                                                           | Built-in transports (files, databases, services).                              | Requires a dedicated Graylog server or cloud instance.                                         |
| Setup Complexity        | Low—straightforward to start using in NestJS or other Node.js frameworks.                                                     | Medium—flexible but requires deeper config, especially for advanced scenarios. | High—must maintain Graylog infrastructure (server, database, etc.).                            |
| Community & Ecosystem   | Growing focus on high-performance Node.js apps, strong contributor base.                                                      | Mature library with extensive community and plugins.                           | Wide enterprise adoption but more of a full-service logging platform.                          |

Summary:

* **Pino**: Best suited for Node.js applications that require high performance, structured JSON logging, and minimal overhead.
* **Winston**: Offers extensive transports and custom formatting but runs slower and requires more complex setup.
* **Graylog**: Primarily a centralized logging platform rather than a direct library. Ideal for enterprise-level aggregation but can be excessive for standard Node.js needs.

## Why we chose Pino

After comparing the three options above, we concluded that Pino aligns best with our requirements for the following reasons:

1. Performance Matters
   * In both frontend SSR and backend contexts, every millisecond counts. Pino’s minimal overhead ensures faster response times and lower latency.
   * NestJS-based services can experience heavy traffic, making a high-performance logger essential to maintain throughput.
2. Seamless Compatibility
   * The [nestjs-pino](https://github.com/iamolegga/nestjs-pino) integration hooks directly into NestJS middleware, providing transparent request and response logging.
   * Pino’s JSON-based approach makes it straightforward to send logs to AWS services like CloudWatch, Lambda, or S3 for centralized management.
   * Its simplicity also ensures it can be used across the diverse Node.js frameworks we employ: NestJS, Angular, Next.js, etc.
3. Developer-Friendly Experience
   * Pino’s development-friendly experience is enhanced with [pino-pretty](https://github.com/pinojs/pino-pretty), a separate dependency that formats logs into clean, human-readable output for debugging.
   * This drastically improves debugging efficiency and lowers the learning curve for team members.
4. Scalability for Distributed Systems
   * For microservices or distributed architectures on AWS, Pino’s structured logs are easier to integrate with OpenTelemetry, AWS X-Ray, or other centralized logging systems.
   * As our projects grow, Pino’s lightweight nature means we can scale without refactoring our logging strategy.

In short, Pino provides the right blend of speed, flexibility, and ease of use—making it the standout choice in our stack.

## Prerequisites

Regardless of the framework, you’ll typically need:

* `pino`: The core library for logging (not required for NestJS apps).
* `pino-pretty` (dev-only): Optional for human-readable server logs in development. Not recommended for production usage since it’s not as performant.

```bash
pnpm i -E pino && pnpm i -D -E pino-pretty
```

## Configuration

### Next.js

Create a `logger.ts` file - it will have Pino instance for both client and server side code:

```ts
// lib/logger/logger.ts

import pino from "pino";

const isBrowser = typeof window !== "undefined";
const isProd = process.env.NODE_ENV === "production";

// Customize log levels for client and server
// In production, typically "info" or "warn" for server logs
// In development, "debug" is helpful
// You can also use custom environment variable like NEXT_PUBLIC_PINO_LOG_LEVEL
const level = isProd ? "info" : "debug";

// For demonstration, we’ll use the same log level for both client and server,
// but you can vary them if needed (e.g., higher level client logs, more detailed server logs).

export const logger = isBrowser
  ? // Client-side config
    pino({
      browser: {
        // This option logs objects in a more readable console format
        asObject: true,
      },
      level,
    })
  : // Server-side config
    pino({
      level,
      // Example of redacting sensitive fields. Adjust to your needs.
      redact: {
        paths: ["req.headers.authorization", "*.password"],
        censor: "[REDACTED]",
      },
      // Use transport or additional options if you want to pipe
      // logs to an external service on the server side
      // e.g., pino.transport({ target: 'pino-pretty' }) for dev only
      transport: isProd ? undefined : { target: "pino-pretty" },
    });
```

Add Pino dependencies to `serverExternalPackages` in `next.config.ts`:

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  serverExternalPackages: ["pino", "pino-pretty"],
};

export default nextConfig;
```

You can now use it in server components and route handlers:

```ts
// app/api/example/route.ts
import { NextRequest, NextResponse } from 'next/server';
import logger from '@/lib/logger/logger'; // adjust path as needed

export async function GET(request: NextRequest) {
  logger.info('Server-side log: GET /api/example');
  // ... your logic here
  return NextResponse.json({ success: true });
}
```

In client components, logs will be shown in a structured format (due to `asObject: true`):

```jsx
// app/home/page.tsx
'use client';

import logger from '@/lib/logger';

export default function HomePage() {
  logger.debug('Client-side debug log: /home');

  return <div>Welcome to the home page</div>;
}
```

### Angular

Create an Angular service that wraps the Pino logger, this allows you to configure and customize behavior for client-side logging.

```ts
// app/services/logger/logger.service.ts

import { Injectable } from '@angular/core';
import pino from 'pino';
import { environment } from '@/environments/environment';

@Injectable({
  providedIn: 'root',
})
export class LoggerService {
  private logger: pino.Logger;

  constructor() {
    this.logger = pino({
      level: environment.logLevel, // Configure log level based on environment
      browser: {
        asObject: true, // Optional: Format logs as objects for easier reading
      },
    });
  }

  // Expose the logger instance directly
  public getLogger(): pino.Logger {
    return this.logger;
  }
}
```

You can now use the logger directly in your components or services by retrieving the instance from `LoggerService`:

```ts
import { Component, OnInit } from '@angular/core';
import { LoggerService } from '@/services/logger/logger.service';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
})
export class HomeComponent implements OnInit {
  private logger = this.loggerService.getLogger();

  constructor(private readonly loggerService: LoggerService) {}

  ngOnInit(): void {
    this.logger.info('HomeComponent initialized');
    this.logger.debug('Debugging details', { userId: 123, action: 'viewedHome' });
  }
}
```

### NestJS

First you need to install required dependencies:

```bash
pnpm i -E nestjs-pino pino-http && pnpm i -D -E pino-pretty
```

In `app.module.ts` import Pino `LoggerModule`:

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        level: process.env.NODE_ENV !== 'production' ? 'debug' : 'info',
        transport:
          process.env.NODE_ENV !== 'production'
            ? { target: 'pino-pretty' }
            : undefined,
      },
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

In `main.ts` file, setup Pino as the default app logger:

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { Logger } from 'nestjs-pino';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { bufferLogs: true });

  app.useLogger(app.get(Logger));

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

You can now use the default Nest `Logger` class, logs will be printed through Pino:

```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';
import { Logger } from '@nestjs/common';

@Controller()
export class AppController {
  private readonly logger = new Logger(AppController.name);

  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    this.logger.log('Hello World');

    return this.appService.getHello();
  }
}
```

## Customizing Pino

Pino’s power comes from its performance and flexibility. Although it’s designed to be simple out of the box, you can adapt it to fit advanced logging needs. Some key customization areas include:

1. Log Levels and Configuration Options
2. Redaction of Sensitive Data
3. Serializers for Custom Fields
4. Child Loggers
5. Hooks and Formatters
6. Transports (Built-In vs. Custom)

You can read more about all of the customization options in [Pino docs](https://getpino.io/#/).

> ⚠️ Warning: The more custom logic you add (especially hooks and synchronous transports), the more performance overhead you introduce - always be mindful.

## Best Practices

1. Use Environment-Based Log Levels
   * Configure log levels (`debug`, `info`, `warn`, etc.) depending on the environment. For example, use `debug` or `trace` in development, and `info` or `warn` in production. This keeps log volumes manageable and ensures meaningful logs in production.
2. Employ Redaction and Serializers
   * Always redact sensitive information (e.g., passwords, tokens) using Pino’s `redact` option.
   * Define serializers to transform or anonymize complex objects before they’re logged, preventing potential leaks of personally identifiable or sensitive data.
3. Leverage Child Loggers
   * Create child loggers for different application modules. This helps organize logs by scope (e.g., an auth logger vs. a payments logger), making it easier to filter and analyze logs in large codebases.
4. Avoid Blocking Transports
   * Write logs to `stdout` by default and rely on external tools (e.g., AWS CloudWatch, ELK stack) for ingestion. If you must use custom transports, ensure they don’t introduce blocking `I/O` or heavy synchronous operations that degrade app performance.
5. Keep Configuration Lean
   * Start with minimal configuration to maintain Pino’s performance edge. Only add hooks, advanced formatters, or complex transports if you truly need them—each feature may increase overhead or complexity.
6. Test and Monitor
   * Regularly review logs in development, staging, and production to ensure they provide actionable insights. Validate that sensitive data is properly redacted and that log levels match your operational needs.
