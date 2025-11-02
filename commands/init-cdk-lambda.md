---
description: Initialize an AWS Lambda project using AWS CDK and TypeScript with Middy middleware and proper logging
allowed-tools: [Bash, Write, Read, TodoWrite, AskUserQuestion]
---

Initialize an AWS Lambda project (CDK, TypeScript) with Middy middleware, @aws-lambda-powertools/logger, and AWS CDK infrastructure.

## Before Starting

1. **Create TodoWrite list** to track initialization progress:
   - Break down the setup into measurable subtasks
   - Mark each subtask as completed when done
   - Track any issues encountered

2. **Gather required information** using AskUserQuestion:
   - Project name (for CDK stack and package.json)
   - AWS region to deploy to
   - Lambda function purpose/description

## Prerequisites Check

Before proceeding, verify:
- Node.js 18+ installed: `node --version`
- AWS CLI configured: `aws sts get-caller-identity`
- CDK CLI installed: `cdk --version` (if not: `npm install -g aws-cdk`)

## Core Requirements

- Lambda function that receives data, logs it, and returns a response
- Middy middleware for input validation and error handling
- Structured logging with @aws-lambda-powertools/logger
- AWS CDK infrastructure as code
- TypeScript for type safety

---

## Step 1: Initialize Project Structure

Execute in order:

```bash
npm init -y
git init
cdk init app --language typescript
```

**Verify**:
- `package.json` exists
- `bin/` and `lib/` directories created
- `.git` directory initialized

---

## Step 2: Install Dependencies

```bash
npm install aws-cdk-lib constructs @aws-lambda-powertools/logger @middy/core @middy/http-json-body-parser @middy/http-error-handler
npm install -D @types/node @types/aws-lambda typescript
```

**Verify**: Check that all packages appear in `package.json` dependencies

---

## Step 3: Create .gitignore

Create `.gitignore` file with:

```
# Dependencies
node_modules/
package-lock.json

# CDK
*.js
*.js.map
*.d.ts
cdk.out/
.cdk.staging/

# Environment
.env
.env.local

# IDE
.vscode/
.idea/

# Build output
dist/
build/

# Logs
*.log
```

---

## Step 4: Create Logger Utility

Create `utils/logger.ts`:

```typescript
import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({
  serviceName: process.env.SERVICE_NAME || 'lambda-service',
  logLevel: process.env.LOG_LEVEL || 'INFO',
});

export { logger };
```

**Verify**: File exists at `utils/logger.ts`

---

## Step 5: Create Lambda Handler

Create `lambda/index.ts`:

```typescript
import middy from '@middy/core';
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import jsonBodyParser from '@middy/http-json-body-parser';
import httpErrorHandler from '@middy/http-error-handler';
import { logger } from '../utils/logger';

interface RequestBody {
  message?: string;
}

const baseHandler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  logger.info('Received event', {
    path: event.path,
    method: event.httpMethod
  });

  const body = event.body as unknown as RequestBody;

  logger.info('Processing request', { body });

  return {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Success!',
      receivedMessage: body?.message || 'No message provided',
      timestamp: new Date().toISOString(),
    }),
  };
};

export const handler = middy(baseHandler)
  .use(jsonBodyParser())
  .use(httpErrorHandler());
```

**Verify**: File exists at `lambda/index.ts` with proper TypeScript types

---

## Step 6: Create CDK Stack

Update `lib/<project-name>-stack.ts` with:

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as nodejs from 'aws-cdk-lib/aws-lambda-nodejs';
import { Construct } from 'constructs';

export class MyLambdaStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Lambda Function
    const myFunction = new nodejs.NodejsFunction(this, 'MyFunction', {
      entry: 'lambda/index.ts',
      handler: 'handler',
      runtime: lambda.Runtime.NODEJS_20_X,
      timeout: cdk.Duration.seconds(30),
      memorySize: 256,
      environment: {
        SERVICE_NAME: 'my-lambda-service',
        LOG_LEVEL: 'INFO',
      },
      bundling: {
        minify: true,
        sourceMap: true,
        target: 'es2020',
        externalModules: ['@aws-sdk/*'],
      },
    });

    // Output the function name
    new cdk.CfnOutput(this, 'FunctionName', {
      value: myFunction.functionName,
      description: 'Lambda Function Name',
    });

    new cdk.CfnOutput(this, 'FunctionArn', {
      value: myFunction.functionArn,
      description: 'Lambda Function ARN',
    });
  }
}
```

**Verify**: Stack file compiles without TypeScript errors

---

## Step 7: Add Build Scripts

Update `package.json` to add scripts section:

```json
{
  "scripts": {
    "build": "tsc",
    "watch": "tsc -w",
    "test": "jest",
    "cdk": "cdk",
    "synth": "cdk synth",
    "deploy": "cdk deploy",
    "destroy": "cdk destroy"
  }
}
```

---

## Step 8: Synthesize and Validate CDK Stack

```bash
npm run build
cdk synth
```

**Verify**:
- No TypeScript compilation errors
- `cdk.out/` directory created
- CloudFormation template generated successfully

---

## Step 9: Deploy to AWS (Optional)

If AWS credentials are configured:

```bash
cdk bootstrap  # Only needed once per account/region
cdk deploy --all --require-approval never
```

**Verify**:
- Stack deployed successfully
- Function name and ARN output displayed
- Check AWS Console Lambda service for the function

---

## Troubleshooting

**If `cdk init` fails**:
- Ensure directory is empty or only contains .git
- Install CDK globally: `npm install -g aws-cdk`

**If `npm install` fails**:
- Clear npm cache: `npm cache clean --force`
- Delete `node_modules` and retry

**If CDK synth fails**:
- Run TypeScript compilation: `npm run build`
- Check for TypeScript errors: `npx tsc --noEmit`

**If deployment fails**:
- Verify AWS credentials: `aws sts get-caller-identity`
- Check IAM permissions for CDK deployment
- Ensure region is specified in AWS config

**If Lambda build fails**:
- Check `lambda/index.ts` syntax
- Verify all imports are correct
- Ensure @types packages are installed

**If Middy middleware errors occur**:
- Verify middleware import syntax (default vs named imports)
- Check middleware execution order
- Ensure event type matches middleware expectations

---

## Success Criteria

✅ CDK project initialized with TypeScript
✅ Lambda handler with Middy middleware configured
✅ Logger utility properly set up with powertools
✅ CDK stack synthesizes without errors (`cdk synth` succeeds)
✅ All TypeScript files compile without errors
✅ Project structure follows AWS best practices
✅ .gitignore properly configured
✅ (Optional) Successfully deployed to AWS

---

## Next Steps

After successful initialization:
1. Add API Gateway if HTTP endpoint is needed
2. Configure DynamoDB tables for data persistence
3. Add IAM roles and policies for AWS service access
4. Implement unit tests for Lambda handlers
5. Set up environment-specific configurations
6. Add CDK Nag for security compliance checks

---

## Project Structure

Your project should now have:

```
.
├── bin/
│   └── <project>.ts          # CDK app entry point
├── lib/
│   └── <project>-stack.ts    # CDK stack definition
├── lambda/
│   └── index.ts              # Lambda handler
├── utils/
│   └── logger.ts             # Logger configuration
├── cdk.out/                  # CDK synthesized output
├── node_modules/
├── .gitignore
├── cdk.json                  # CDK configuration
├── package.json
├── tsconfig.json             # TypeScript configuration
└── README.md
```
