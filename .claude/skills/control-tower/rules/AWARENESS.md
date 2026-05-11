## Awareness: Lessons Learned

Hard-won lessons from implementation. Read before writing integration code.

---

### Cross-Service Contracts

#### Always verify actual payload shapes — never assume field names

**Incident:** Gateway auth middleware used `token_type` and `user_id` to parse JWTs, but the Auth Service issues tokens with `type` and `sub`. The gateway returned 401 on every authenticated request.

**Rule:** Before writing code that consumes another service's output (JWT claims, gRPC responses, event payloads), read the **producing** service's design doc or source code to confirm exact field names. Do not guess or infer from convention.

**Applies to:** JWT claims, gRPC response fields, Redis event payloads, any cross-service data contract.

---

### Database

#### Match DB schema to ORM model — never create tables manually

**Incident:** A manually-written `CREATE TABLE` used `VARCHAR` for the `id` column, but the SQLAlchemy ORM model declares `UUID(as_uuid=False)`. PostgreSQL rejected queries with `operator does not exist: character varying = uuid`.

**Rule:** Always use `Base.metadata.create_all(engine)` or Alembic migrations to create tables. Never write raw DDL unless you have verified every column type against the ORM model. If no migrations exist, generate them from the model.

---

### Dependencies

#### Pin transitive dependencies that break on major versions

**Incident:** `passlib[bcrypt]` silently pulled `bcrypt>=5.0` which changed its API, causing `ValueError` at runtime. No test caught this because the version wasn't pinned.

**Rule:** When a library wraps another library (passlib wraps bcrypt), either pin the transitive dependency (`bcrypt>=4.0,<5.0`) or use the underlying library directly. Prefer direct usage over wrappers to reduce version coupling.

#### `[tool.uv.sources]` is invisible to pip — install local packages explicitly in Dockerfiles

**Incident:** Auth Service's `pyproject.toml` declared `events` with `[tool.uv.sources] events = { path = "../../lib/events" }`. Docker's `pip install .` ignored this uv-specific config and installed the wrong `Events` package from PyPI.

**Rule:** In Dockerfiles, explicitly `pip install /path/to/local/package` before `pip install .` for any dependency that uses `[tool.uv.sources]` local paths.

---

### Code Generation (Proto / gRPC)

#### Generated code needs import path fixups — automate it

**Incident:** `buf generate` / `grpc_tools.protoc` produces stubs with `from auth.v1 import auth_pb2`, but services import as `from proto.auth.v1 import auth_pb2`. Every generated file needed a sed fixup.

**Rule:** Generated code import paths rarely match your project layout. Always automate the fixup in both:
- `scripts/generate-proto.sh` (local development)
- `Dockerfile` (container builds)

Use `sed -i.bak` (not `sed -i ''`) for cross-platform compatibility (macOS + Linux).

#### Proto stubs are gitignored — each build must self-generate

**Incident:** Docker containers crashed because proto stubs only existed on the developer's host machine. The stubs were in `.gitignore` and never committed.

**Rule:** Every Dockerfile that needs gRPC stubs must generate them during build:
```dockerfile
COPY proto /app/proto          # bring in .proto source
RUN python -m grpc_tools.protoc ... # generate stubs
```
Never rely on host-generated files being present in the Docker build context.

---

### Service Startup

#### Services need startup readiness — add retry logic for dependent fetches

**Incident:** The gateway fetched JWKS from the Auth Service during startup, but the Auth Service hadn't finished booting yet. The gateway crashed immediately.

**Rule:** Any startup dependency on another service (JWKS fetch, gRPC health probe, DB migration check) must have a retry loop with backoff. `depends_on: service_started` only guarantees the container started, not that the application is ready.
