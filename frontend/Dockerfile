# Stage 1: Build React app
FROM node:18

WORKDIR /app

COPY frontend/package*.json ./
RUN npm install --ignore-scripts

COPY frontend .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
COPY --from=0 /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
