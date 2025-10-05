# Build stage
FROM golang:1.23-alpine AS builder
RUN apk add --no-cache git ca-certificates
WORKDIR /workspace
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o rest-api .

# Final image
FROM gcr.io/distroless/base-debian11:latest
WORKDIR /app
COPY --from=builder /workspace/rest-api /app/rest-api

# Note: Mount crypto materials at runtime for security
# DO NOT copy organizations folder into the image in production

EXPOSE 8080
ENTRYPOINT ["/app/rest-api"]