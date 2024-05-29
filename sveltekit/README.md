# Dockerfiles for SvelteKit projects

## Dockerfile.1

```Dockerfile
FROM node:20.12-alpine3.19 AS builder
```

This line selecting NodeJS on Alpine. Alpine is very small and secure.

It is recommended to use a two-stage build process because it results in a smaller image. This is achieved by leaving unnecessary files, such as node_modules, behind in the build stage and only copying the essential application code and dependencies into the final image.

```Dockerfile
RUN npm install -g pnpm
```

Because I'm always using pnpm (and you should too), this line installing pnpm before everything else.

```Dockerfile
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm i
```

Selecting working directory /app.
Copying package.json and pnpm-lock.yaml into /app and then installing packages. If you are using npm, it should be package-lock.json.

By copying only package.json and installing dependencies before copying the entire project, Docker can cache packages. This reduces installation time subsequently, as packages are only reinstalled when the list of dependencies in package.json changes.

```Dockerfile
COPY . .
RUN npm run build
```

This copies the entire project to the /app directory, and then runs the build process.

```Dockerfile
FROM node:20.12.0-alpine3.19
WORKDIR /app
COPY --from=builder /app/build build/
COPY package.json .
EXPOSE 3000

ENV NODE_ENV=production

CMD ["node", "build"]
```

In the second build stage, the same Node.js image from the first stage is used. This stage copies the built project from the first stage, excluding unnecessary files like node_modules.

SvelteKit automatically opens port 3000, as documented here.

It's important to remember that ENV variables set in Dockerfile are <b>runtime</b> variables (process.env.\<var>). This means that they must be set in the final stage where the aplication is started. 
The variables stored in .env file in SvelteKit projects are used and set during build time.

Finally, NodeJS runs index.js from the build directory.
