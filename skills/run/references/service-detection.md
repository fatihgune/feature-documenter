# Service Detection Patterns

Detection is two-phase:
1. **File-glob scan** (fast, no file reading): Check for marker files and directory patterns
2. **Content scan** (targeted reads): Confirm stack and extract routes/handlers from candidates found in phase 1

## Spring Boot (Java/Kotlin)

### Detection Signals

Glob for: `pom.xml`, `build.gradle`, `build.gradle.kts`
Confirm: file contains `spring-boot-starter-web` or `spring-boot-starter-webflux`
Secondary: presence of `src/main/java` or `src/main/kotlin` directory

### Routes/Endpoints

Glob for: `**/*Controller.java`, `**/*Controller.kt`, `**/*Resource.java`, `**/*Resource.kt`
Annotations to scan for: `@RestController`, `@Controller`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`
Extract: HTTP method + path from annotation value, class-level `@RequestMapping` prefix

### Service/Business Logic

Glob for: `**/*Service.java`, `**/*Service.kt`, `**/*ServiceImpl.java`, `**/*ServiceImpl.kt`
Annotations: `@Service`, `@Component`, `@Transactional`
Follow: constructor injection parameters to identify dependencies

### Outbound Calls

Patterns:
- `RestTemplate` / `WebClient` / `FeignClient` usage
- `@FeignClient(name = "service-name")` annotations
- `KafkaTemplate.send`, `RabbitTemplate.convertAndSend`, `JmsTemplate.send`
- gRPC stubs: classes ending in `Grpc`, `BlockingStub`, `FutureStub`

### Background Jobs/Events

Patterns:
- `@Scheduled` annotation (cron jobs)
- `@KafkaListener`, `@RabbitListener`, `@JmsListener` (event consumers)
- `ApplicationEventPublisher.publishEvent` (internal events)
- `@EventListener`, `@TransactionalEventListener`
- `@Async` annotated methods

## Express (Node.js)

### Detection Signals

Glob for: `package.json`
Confirm: dependencies include `express`
Secondary: presence of `app.js`, `server.js`, `index.js`, or `src/app.ts`

### Routes/Endpoints

Glob for: `**/routes/**/*.{js,ts}`, `**/router/**/*.{js,ts}`, `**/controllers/**/*.{js,ts}`
Patterns: `router.get(`, `router.post(`, `router.put(`, `router.delete(`, `router.patch(`
Also: `app.get(`, `app.post(`, etc.
Extract: HTTP method + path string from first argument

### Service/Business Logic

Glob for: `**/services/**/*.{js,ts}`, `**/handlers/**/*.{js,ts}`, `**/middleware/**/*.{js,ts}`
Follow: require/import chains from route handlers

### Outbound Calls

Patterns:
- `axios.get`, `axios.post`, `fetch(`, `got(`, `request(`
- `amqplib` channel.publish, channel.sendToQueue
- `kafkajs` producer.send
- gRPC client calls

### Background Jobs/Events

Patterns:
- `bull` / `bullmq` queue definitions and processors
- `node-cron` / `cron` scheduled tasks
- `agenda` job definitions
- EventEmitter patterns with named events
- `amqplib` channel.consume (message consumers)

## NestJS (Node.js)

### Detection Signals

Glob for: `package.json`
Confirm: dependencies include `@nestjs/core`
Secondary: presence of `nest-cli.json` or `src/main.ts` with `NestFactory`

### Routes/Endpoints

Glob for: `**/*.controller.ts`, `**/*.controller.js`
Decorators: `@Controller()`, `@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`
Extract: path from decorator argument, class-level `@Controller('prefix')` prefix

### Service/Business Logic

Glob for: `**/*.service.ts`, `**/*.service.js`
Decorators: `@Injectable()`
Follow: constructor injection parameters

### Outbound Calls

Patterns:
- `HttpService` / `HttpModule` (wraps axios)
- `ClientProxy.send`, `ClientProxy.emit` (microservice calls)
- `@MessagePattern`, `@EventPattern` on the receiving side
- gRPC `@GrpcMethod` decorator

### Background Jobs/Events

Patterns:
- `@Cron()` decorator from `@nestjs/schedule`
- `@Process()` decorator from `@nestjs/bull`
- `EventEmitter2` from `@nestjs/event-emitter`
- `@OnEvent()` decorator

## Django (Python)

### Detection Signals

Glob for: `manage.py`, `settings.py`, `**/settings/*.py`
Confirm: `INSTALLED_APPS` in settings, `django` in requirements.txt or Pipfile
Secondary: `wsgi.py` or `asgi.py`

### Routes/Endpoints

Glob for: `**/urls.py`, `**/views.py`, `**/views/**/*.py`, `**/viewsets.py`
Patterns: `path(`, `re_path(`, `url(` in urls.py
DRF: `@api_view`, `class *ViewSet`, `class *APIView`, router.register
Extract: URL pattern + view function/class reference

### Service/Business Logic

Glob for: `**/services.py`, `**/services/**/*.py`, `**/utils.py`, `**/tasks.py`
Follow: imports from view functions
Models: `**/models.py`, `**/models/**/*.py`

### Outbound Calls

Patterns:
- `requests.get`, `requests.post`, `httpx.get`, `httpx.post`
- `pika` (RabbitMQ) channel.basic_publish
- `kafka-python` producer.send
- gRPC stub calls

### Background Jobs/Events

Patterns:
- `@shared_task`, `@app.task` (Celery tasks)
- `@periodic_task`, Celery beat schedule in settings
- Django signals: `@receiver`, `Signal.send`
- `post_save`, `pre_save`, `post_delete` signal connections

## FastAPI (Python)

### Detection Signals

Glob for: `requirements.txt`, `pyproject.toml`, `Pipfile`
Confirm: `fastapi` in dependencies
Secondary: `main.py` or `app.py` with `FastAPI()` instantiation

### Routes/Endpoints

Glob for: `**/routers/**/*.py`, `**/routes/**/*.py`, `**/api/**/*.py`, `main.py`, `app.py`
Decorators: `@app.get(`, `@app.post(`, `@router.get(`, `@router.post(`, etc.
Extract: HTTP method + path from decorator

### Service/Business Logic

Glob for: `**/services/**/*.py`, `**/crud/**/*.py`, `**/handlers/**/*.py`
Follow: function calls from route handlers, dependency injection via `Depends()`

### Outbound Calls

Patterns:
- `httpx.AsyncClient`, `aiohttp.ClientSession`
- `requests.get`, `requests.post`
- Message broker clients (same as Django)
- gRPC stub calls

### Background Jobs/Events

Patterns:
- `@app.on_event("startup")`, `@app.on_event("shutdown")`
- Celery tasks (same as Django)
- `BackgroundTasks.add_task` (FastAPI built-in)
- `arq` worker definitions

## Go (net/http + gRPC)

### Detection Signals

Glob for: `go.mod`
Confirm: presence of `go.mod`
Secondary: `main.go` or `cmd/*/main.go`

### Routes/Endpoints

Glob for: `**/*.go`
Patterns:
- `http.HandleFunc(`, `mux.HandleFunc(`, `r.HandleFunc(` (stdlib/gorilla)
- `e.GET(`, `e.POST(` (Echo)
- `router.GET(`, `router.POST(` (Gin)
- `.Get(`, `.Post(` (Chi)
- gRPC: `pb.Register*Server(` function calls
- gRPC service definitions in `**/*.proto` files

### Service/Business Logic

Go does not have a standard service layer convention. Look for:
- Packages named `service`, `handler`, `usecase`, `domain`
- Struct methods called from HTTP handlers
- Interface definitions that handlers depend on

### Outbound Calls

Patterns:
- `http.Get(`, `http.Post(`, `http.NewRequest(`
- `client.Do(`, where client is `*http.Client`
- gRPC: `pb.New*Client(` dial connections
- NATS: `nc.Publish(`, `nc.Subscribe(`
- Kafka: `writer.WriteMessages(`, `producer.Produce(`

### Background Jobs/Events

Patterns:
- Goroutines in `init()` or `main()` with tickers/timers
- `cron.New()` from robfig/cron
- NATS/Kafka consumers in long-running goroutines
- `go func()` patterns with channel consumption

## Rails (Ruby)

### Detection Signals

Glob for: `Gemfile`, `config/routes.rb`
Confirm: `rails` gem in Gemfile
Secondary: `config/application.rb` with `Rails::Application`

### Routes/Endpoints

Glob for: `config/routes.rb`, `config/routes/**/*.rb`
Patterns: `get`, `post`, `put`, `patch`, `delete`, `resources`, `resource`, `namespace`, `scope`
Controllers: `app/controllers/**/*_controller.rb`
Extract: route definitions map to controller#action

### Service/Business Logic

Glob for: `app/services/**/*.rb`, `app/interactors/**/*.rb`, `app/operations/**/*.rb`
Models: `app/models/**/*.rb`
Follow: controller action -> service/model calls

### Outbound Calls

Patterns:
- `HTTParty.get`, `Faraday.get`, `Net::HTTP`
- `RestClient.get`, `RestClient.post`
- `Bunny` (RabbitMQ) channel.publish
- `Karafka` producer
- gRPC stub calls

### Background Jobs/Events

Patterns:
- `ActiveJob` subclasses, `perform_later`, `perform_now`
- `Sidekiq::Worker`, `include Sidekiq::Job`
- `ActiveSupport::Notifications`
- `after_commit` callbacks
- `config/sidekiq.yml` for scheduled jobs

## ASP.NET Core (.NET)

### Detection Signals

Glob for: `*.csproj`, `*.sln`
Confirm: `Microsoft.AspNetCore` or `Microsoft.NET.Sdk.Web` in .csproj
Secondary: `Program.cs` or `Startup.cs`

### Routes/Endpoints

Glob for: `**/*Controller.cs`, `**/Controllers/**/*.cs`
Attributes: `[ApiController]`, `[Route]`, `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`
Minimal APIs: `app.MapGet(`, `app.MapPost(` in Program.cs
Extract: route template from attribute or MapX call

### Service/Business Logic

Glob for: `**/*Service.cs`, `**/Services/**/*.cs`, `**/*Handler.cs`
Follow: constructor injection via DI container
MediatR: `IRequestHandler<TRequest, TResponse>` implementations

### Outbound Calls

Patterns:
- `HttpClient.GetAsync`, `HttpClient.PostAsync`, `IHttpClientFactory`
- `MassTransit` IBus.Publish, ISendEndpoint.Send
- `NServiceBus` IMessageSession.Send
- gRPC client calls
- `Refit` interface-based HTTP clients

### Background Jobs/Events

Patterns:
- `IHostedService`, `BackgroundService` subclasses
- `Hangfire` `BackgroundJob.Enqueue`, `RecurringJob.AddOrUpdate`
- `MediatR` notifications: `INotificationHandler<T>`
- `IObserver<T>` patterns
- Quartz.NET `IJob` implementations
