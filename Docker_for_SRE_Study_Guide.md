# Docker cho SRE — tài liệu tự học từ nền tảng đến vận hành thực tế

> Mục tiêu: sau khi học xong, bạn có thể đóng gói một ứng dụng thành image, chạy một hệ multi-container bằng Docker Compose, lưu dữ liệu đúng cách, quan sát và debug sự cố, đồng thời review được các rủi ro production phổ biến.

## Lộ trình học

Tài liệu đi theo hành trình: **hiểu Docker → đóng gói ứng dụng → chạy container → ghép nhiều service → vận hành → đưa vào production**.

1. **Hiểu Docker**: Docker, container, image, layer và kiến trúc Docker.
2. **Đóng gói ứng dụng**: Dockerfile, build context, cache và multi-stage build.
3. **Chạy một container**: vòng đời, port, cấu hình runtime và resource limits.
4. **Chạy hệ nhiều container**: storage, networking và Docker Compose.
5. **Vận hành như SRE**: healthcheck, logs, observability và debug.
6. **Đưa vào production**: security, registry, CI/CD, checklist và các bài lab.

## Cách đọc tài liệu

- **Trọng tâm**: kiến thức phải hiểu chắc, thường được hỏi khi phỏng vấn và là nền tảng cho Kubernetes.
- **Dùng thực tế**: thao tác xuất hiện thường xuyên trong công việc DevOps/SRE.
- **Nâng cao**: nên học sau khi đã làm các lab cuối tài liệu.
- Các lệnh có thể khác đôi chút giữa Linux, macOS và Windows/WSL. Trong môi trường production, hãy kiểm tra phiên bản Docker Engine và chính sách nội bộ trước khi chạy lệnh thay đổi/xóa dữ liệu.

---

# Phần 1 — Hiểu Docker

## 1.1. Docker là gì?

Docker là một nền tảng để build, phân phối và chạy ứng dụng dưới dạng **container**. Thay vì cài runtime và thư viện trực tiếp lên từng máy, ta đóng gói ứng dụng cùng các dependency thành một **image**. Từ image đó, Docker tạo ra một hoặc nhiều container.

Một container là một process được cô lập tương đối với các process khác. Trên Linux, sự cô lập chủ yếu dựa vào kernel namespaces; cgroups dùng để theo dõi/giới hạn tài nguyên. Container không có một kernel riêng giống máy ảo.

### Khi nào dùng?

- Chuẩn hóa môi trường local, CI và production.
- Đóng gói microservice ( Microservice là cách xây dựng ứng dụng thành nhiều dịch vụ nhỏ, mỗi dịch vụ làm một chức năng riêng và có thể triển khai độc lập), web API, worker, database cho môi trường phát triển/test.
- Tạo artifact có version để triển khai và rollback.
- Chạy workload trên orchestrator như Kubernetes.

### Khi nào Docker không tự giải quyết được vấn đề?

- Docker không tự cung cấp HA, autoscaling, multi-host scheduling hay service mesh.
- Container không biến ứng dụng stateful thành stateless.
- Container không thay thế backup, monitoring, secrets manager hoặc quy trình incident response.
- Isolation của container không nên mặc định được coi mạnh như ranh giới VM.

**Trọng tâm:** Docker giải quyết bài toán đóng gói và runtime. Production reliability vẫn cần thiết kế ứng dụng và hệ thống vận hành.

Nguồn: [Docker overview](https://docs.docker.com/get-started/docker-overview/), [What is a container?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)

---

## 1.2. Container và máy ảo khác nhau thế nào?

| Khía cạnh | Container | Máy ảo |
|---|---|---|
| Kernel | Chia sẻ kernel của host | Mỗi VM có guest OS/kernel riêng |
| Đơn vị chính | Process + filesystem + runtime config | Một máy tính ảo hoàn chỉnh |
| Khởi động | Thường nhanh | Thường chậm hơn |
| Mật độ | Cao hơn | Thấp hơn vì overhead guest OS |
| Isolation | Process/kernel primitives | Ranh giới hypervisor/VM |
| Cách dùng phổ biến | Đóng gói và chạy ứng dụng | Ranh giới hạ tầng, tenancy, OS khác nhau |

Trong thực tế cloud, hai mô hình thường dùng cùng nhau: VM là worker/host, còn container chạy bên trong VM.

**Trọng tâm:** container là process, không phải một “máy Linux mini”. Khi process PID 1 kết thúc, container dừng.

---

## 1.3. Kiến trúc Docker

Các thành phần chính:

1. **Docker CLI**: nơi bạn chạy `docker build`, `docker run`, `docker logs`…
2. **Docker daemon/Engine**: quản lý image, container, network và volume trên host.
3. **Registry**: nơi lưu và phân phối image, ví dụ Docker Hub hoặc registry nội bộ.
4. **Container runtime**: tạo và chạy container dựa trên image cùng runtime configuration.

![alt text](image.png)
Client gửi lệnh → Docker daemon xử lý → lấy/tạo image → tạo container để chạy ứng dụng.

Luồng điển hình:

```text
Git source
  -> Dockerfile
  -> docker build
  -> image
  -> push registry
  -> pull/deploy
  -> container + env + mount + network + limits
```

### Tại sao SRE phải phân biệt image và runtime config?

Cùng một image có thể chạy tốt ở staging nhưng lỗi ở production do biến môi trường sai, volume thiếu, DNS không hoạt động, port không publish, thiếu memory hoặc secret hết hạn. Vì vậy khi debug phải kiểm tra cả hai phía.

---

## 1.4. Image, container và layer

### Image là gì?

Image là một gói chuẩn chứa filesystem và metadata cần để chạy ứng dụng: binary, runtime, thư viện, cấu hình mặc định và command. Image có tính **immutable** (bất biến): muốn thay đổi, ta build image mới.

Image gồm nhiều **layer**. Mỗi instruction thích hợp trong Dockerfile tạo ra thay đổi filesystem/metadata. Docker có thể tái sử dụng layer để giảm thời gian build và dung lượng truyền.

### Container là gì?

Container là “bản chạy cụ thể” được tạo từ image, có thể đang chạy hoặc đã dừng.

Image chỉ đọc, còn khi tạo container Docker thêm một **writable layer** ở trên để lưu các thay đổi lúc chạy, như file app tạo ra hoặc dữ liệu tạm.

Khi chạy, Docker cũng áp dụng cấu hình như:

- Biến môi trường (`ENV`)
- Map port
- Mount volume
- Network
- Giới hạn CPU/RAM
- Cấu hình bảo mật

Xóa container thì các thay đổi ở writable layer cũng mất, trừ dữ liệu đã lưu trong volume/mount.
### Điều dễ nhầm

- Xóa container không xóa image.
- Stop container không xóa container.
- Dữ liệu trong writable layer có thể mất khi container bị xóa/recreate.
- `EXPOSE 8000` không tự publish port 8000 ra host.
- Tag như `latest` không đảm bảo nội dung image bất biến.

**Dùng thực tế:** khi một deployment khác hành vi dù “cùng tag”, kiểm tra image digest (mã định danh), không chỉ nhìn tag. 

Nguồn: [What is an image?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)

---

# Phần 2 — Đóng gói ứng dụng

## 2.1. Dockerfile

Dockerfile là file văn bản mô tả cách build image.

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

USER 10001
EXPOSE 8000
CMD ["python", "app.py"]
```

### Các instruction quan trọng

#### `FROM`

Chọn base image. Dùng image từ nguồn tin cậy, kích thước phù hợp và được cập nhật. “Nhỏ nhất” không luôn là “tốt nhất”: image quá tối giản có thể khó debug hoặc không tương thích thư viện.

#### `WORKDIR`

Thiết lập thư mục làm việc cho các instruction tiếp theo và process runtime.

#### `COPY`

Copy file từ build context vào image. Thường ưu tiên `COPY` thay vì `ADD` nếu không cần hành vi đặc biệt.

#### `RUN`

Chạy lệnh trong giai đoạn build và tạo layer. Ví dụ cài OS package hoặc Python dependency.

#### `ENV`

Tạo environment variable tồn tại trong image và container mặc định. Không dùng để chứa secret.

#### `ARG`

Biến dùng trong build. `ARG` cũng không phải nơi an toàn để truyền secret vì có thể xuất hiện trong metadata/history hoặc cache; dùng BuildKit secret mount cho build secret.

#### `EXPOSE`

Mô tả port mà ứng dụng dự kiến lắng nghe. Nó không mở firewall và không thay thế `-p`/`ports`.

#### `CMD` và `ENTRYPOINT`

- `ENTRYPOINT`: executable chính khó bị thay thế hơn.
- `CMD`: command/arguments mặc định, dễ override khi `docker run`.
- Dạng JSON/exec như `CMD ["python", "app.py"]` thường giúp signal đi trực tiếp tới process chính tốt hơn dạng shell.

#### `USER`

Chuyển sang UID/GID không phải root cho runtime. Đây là một trong những hardening control có giá trị cao.

### Build image

```bash
docker build -t my-api:1.0.0 .
docker image ls
docker history my-api:1.0.0
```

### Kiểm tra Dockerfile

```bash
docker build --check .
```

Nguồn: [Dockerfile overview](https://docs.docker.com/build/concepts/dockerfile/), [Dockerfile reference](https://docs.docker.com/reference/dockerfile/), [Build checks](https://docs.docker.com/reference/build-checks/)

---

## 2.2. Build context, `.dockerignore` và cache

Build context là tập file Docker gửi cho builder. Context quá lớn làm build chậm và có nguy cơ đưa file nhạy cảm vào quá trình build.

Ví dụ `.dockerignore`:

```gitignore
.git
.env
node_modules
__pycache__
*.log
dist
coverage
```

### Tối ưu cache bằng thứ tự instruction

Không tốt:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Mỗi lần bất kỳ source file nào đổi, layer cài dependency bị invalidate.

Tốt hơn:

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

Khi chỉ code đổi mà dependency file không đổi, Docker có thể reuse layer cài dependency.

### Lưu ý

- Cache tăng tốc build nhưng không phải bằng chứng rằng dependency đang mới nhất.
- `--no-cache` buộc chạy lại build step nhưng không tự pull base image mới.
- `--pull` yêu cầu lấy base image mới hơn nếu có.
- Với yêu cầu reproducibility cao, pin base image bằng version hoặc digest; đồng thời có quy trình cập nhật tự động để không bị “đóng băng” lỗ hổng.

Nguồn: [Building best practices](https://docs.docker.com/build/building/best-practices/)

---

## 2.3. Multi-stage build

Multi-stage build dùng nhiều `FROM` trong một Dockerfile. Stage đầu build/test; stage cuối chỉ lấy artifact cần chạy.

```dockerfile
FROM node:24-alpine AS build
WORKDIR /src
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /src/dist /usr/share/nginx/html
USER nginx
```

### Khi nào dùng?

- Go/Rust/Java/.NET cần compiler hoặc build tool nhưng runtime không cần.
- Frontend build thành static files.
- Muốn giữ test/debug tools ngoài production image.

### Lợi ích thực tế

- Image nhỏ hơn, pull và rollout nhanh hơn.
- Giảm số package và attack surface.
- Phân tách rõ build-time và run-time dependency.
- Có thể tạo stage `test`, `debug`, `production` cho các mục đích khác nhau.

Nguồn: [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)

---

# Phần 3 — Chạy một container

## 3.1. Vòng đời và các lệnh container cốt lõi

### Chạy container đầu tiên

```bash
docker run -d --name web -p 8080:80 nginx:alpine
```

Giải thích:

- `-d`: chạy nền.
- `--name web`: đặt tên ổn định để thao tác dễ hơn.
- `-p 8080:80`: host port 8080 chuyển tới container port 80.
- `nginx:alpine`: image và tag.

### Bộ lệnh dùng thường xuyên

```bash
docker ps                    # container đang chạy
docker ps -a                 # gồm cả container đã dừng
docker logs -f web           # theo dõi log
docker inspect web           # metadata đầy đủ
docker stats --no-stream     # snapshot tài nguyên
docker top web               # process trong container
docker exec -it web sh       # chạy shell để debug
docker stop web              # dừng graceful
docker rm web                # xóa container
```

### `docker run` thực hiện gì?

Nếu image chưa có local, Docker pull image; sau đó create container, cấu hình filesystem/network/mount, rồi start process chính.

### Thói quen SRE

1. Thu thập `ps`, `logs`, `inspect`, `stats` trước.
2. Chỉ `exec` khi cần xác minh giả thuyết.
3. Không sửa file thủ công trong container như một bản vá lâu dài.
4. Sửa Dockerfile/config trong Git, build image mới và recreate container.

---

## 3.2. Port, environment, command và restart policy

### Port mapping

```bash
docker run -p 127.0.0.1:8080:8000 my-api:1.0.0
```

Cú pháp: `[host_ip:]host_port:container_port`. Bind vào `127.0.0.1` khi chỉ cần truy cập local; bind tất cả interface có thể làm service lộ ra mạng ngoài.

### Environment variables

```bash
docker run --env APP_ENV=prod --env-file .env my-api:1.0.0
```

Environment phù hợp với config không nhạy cảm hoặc secret được inject qua cơ chế quản lý. Không commit `.env` chứa secret; tránh log toàn bộ environment.

### Override command

```bash
docker run --rm my-api:1.0.0 python -m pytest
```

Hữu ích cho one-off job hoặc test, nhưng production command nên được quản lý nhất quán.

### Restart policy

```bash
docker run --restart unless-stopped ...
```

Restart policy hữu ích trên một host đơn, nhưng không thay thế orchestration, readiness, rollout và failover.

---

## 3.3. Resource limits

Mặc định container có thể sử dụng tài nguyên host nếu không bị giới hạn. Một memory leak hoặc traffic spike có thể ảnh hưởng container khác và Docker daemon.

```bash
docker run -d \
  --memory 512m \
  --cpus 1.0 \
  --pids-limit 200 \
  --ulimit nofile=65535:65535 \
  my-api:1.0.0
```

### Memory

- Đặt limit quá thấp có thể gây OOM kill.
- Kiểm tra `docker inspect` để xem `OOMKilled` và exit code.
- Theo dõi working set/RSS, không chỉ average.
- Alert trước khi chạm trần để có thời gian xử lý.

### CPU

CPU limit thường gây throttling, biểu hiện bằng latency tăng thay vì process chết. Theo dõi saturation và tail latency.

### PID và file descriptor

Process leak/fork bomb làm cạn PID; quá nhiều connection/file có thể chạm `nofile`. Limit cần dựa trên workload test và observability.

**Dùng thực tế:** resource request/capacity planning ở Kubernetes kế thừa trực tiếp mental model này.

Nguồn: [Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)

---

# Phần 4 — Chạy ứng dụng nhiều container

## 4.1. Storage: volume, bind mount và tmpfs

### Vì sao cần mount?

Writable layer gắn với lifecycle container. Database, upload, queue state hoặc artifact cần giữ không nên chỉ nằm ở đó.

### Named volume

```bash
docker volume create pgdata
docker run -d --name db \
  --mount type=volume,src=pgdata,dst=/var/lib/postgresql/data \
  postgres:17
```

Docker quản lý volume; dữ liệu tồn tại ngoài lifecycle container. Đây thường là lựa chọn mặc định tốt cho dữ liệu do container tạo.

### Bind mount

```bash
docker run --mount type=bind,src="$PWD",dst=/app my-api:dev
```

Bind một đường dẫn host vào container. Phù hợp để mount source code khi phát triển hoặc đưa file config cụ thể vào container. Nó phụ thuộc cấu trúc host và có thể tạo rủi ro quyền truy cập.

### tmpfs

Tmpfs giữ dữ liệu trong memory và mất khi container dừng. Phù hợp cho dữ liệu tạm hoặc nhạy cảm không cần persist.

### `--mount` hay `-v`?

`--mount` dài hơn nhưng rõ ràng và hỗ trợ đầy đủ option; phù hợp runbook và production script. `-v` ngắn, tiện khi thao tác nhanh.

### Backup và restore

Volume tồn tại không có nghĩa dữ liệu đã được backup. Cần xác định:

- Cơ chế snapshot/logical backup.
- Retention và encryption.
- Restore test định kỳ.
- RPO/RTO.
- Quyền sở hữu file và UID/GID sau restore.

**Cảnh báo:** `docker compose down -v` xóa named volume do Compose quản lý. Không dùng máy móc trên dữ liệu cần giữ.

Nguồn: [Volumes](https://docs.docker.com/engine/storage/volumes/), [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)

---

## 4.2. Docker networking

### Mô hình đơn giản

- Container có network namespace riêng.
- Container nối vào một hoặc nhiều Docker network.
- User-defined network hỗ trợ service discovery bằng container/service name.
- `EXPOSE` mô tả port; `-p` publish port từ host vào container.

### Tạo network

```bash
docker network create app-net

docker run -d --network app-net --name db postgres:17
docker run -d --network app-net --name api -p 8080:8000 my-api:1.0.0
```

Từ `api`, hostname `db` được resolve trên network đó. Không hard-code IP container vì IP có thể đổi sau recreate.

### Debug network

```bash
docker inspect api
docker network inspect app-net
docker exec api getent hosts db
docker exec api sh -c 'nc -vz db 5432'
```

Nếu image tối giản không có công cụ debug, ưu tiên container chẩn đoán riêng trên cùng network thay vì cài tool trực tiếp vào container production.

### Lỗi phổ biến

- App listen trên `127.0.0.1` bên trong container thay vì `0.0.0.0`.
- Nhầm `host_port` và `container_port`.
- Dùng `localhost` để gọi container khác; `localhost` luôn chỉ chính network namespace hiện tại.
- Publish database ra host dù chỉ API cần truy cập.
- Firewall/security group host không cho lưu lượng.

Nguồn: [Networking overview](https://docs.docker.com/engine/network/)

---

## 4.3. Docker Compose

Compose mô tả một ứng dụng multi-container bằng YAML: services, networks, volumes, configs và runtime settings.

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

### Các lệnh quan trọng

```bash
docker compose config              # render/validate cấu hình
docker compose up -d --build       # build và chạy
docker compose ps                  # trạng thái
docker compose logs -f api         # log service api
docker compose exec api sh         # debug
docker compose restart api         # restart một service
docker compose down                # dừng/xóa container + network
```

### `depends_on` không phải phép màu

Thứ tự start container không chứng minh dependency đã sẵn sàng nhận request. Dùng healthcheck hợp lý và ứng dụng vẫn cần retry với backoff khi gọi DB/upstream.

### Khi nào dùng Compose?

- Local development có nhiều service.
- Integration test/CI environment.
- Demo hoặc workload nhỏ trên một host.
- Tài liệu hóa topology và dependency.

### Compose trong production

Compose có thể dùng cho production một host nếu yêu cầu phù hợp, nhưng cần bổ sung override cấu hình, restart policy, resource limit, logging, backup, secrets và quy trình deploy/rollback. Với nhu cầu multi-host, HA hoặc autoscaling, thường cần orchestrator.

Nguồn: [Docker Compose](https://docs.docker.com/compose/), [Compose services reference](https://docs.docker.com/reference/compose-file/services/), [Use Compose in production](https://docs.docker.com/compose/how-tos/production/)

---

# Phần 5 — Vận hành như SRE

## 5.1. Healthcheck, logs và observability

### Running khác healthy

Docker biết process chính còn chạy, nhưng không tự biết API có kết nối DB được hay có phục vụ request đúng không.

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

Healthcheck tốt nên:

- Nhanh, ổn định và có timeout.
- Kiểm tra đúng điều kiện để container phục vụ.
- Không gây tải đáng kể.
- Tránh dependency chain quá sâu khiến một lỗi nhỏ đánh dấu cả hệ thống unhealthy.

### Logging

Ứng dụng container nên ghi log ra stdout/stderr. Log nên có:

- Timestamp có timezone rõ ràng.
- Severity.
- Service/version/environment.
- Request ID hoặc trace ID.
- Error type và stack trace cần thiết.

Không log token, password, session secret hoặc PII không cần thiết. Log local của container không đủ cho production; cần forwarding/collection tập trung và retention phù hợp.

### Ba tín hiệu khác nhau

- **Health**: dùng để ra quyết định restart/routing.
- **Logs**: giải thích sự kiện cụ thể.
- **Metrics/traces**: phát hiện xu hướng, saturation và đường đi request.

Nguồn: [Dockerfile HEALTHCHECK](https://docs.docker.com/reference/dockerfile/#healthcheck), [Docker logging](https://docs.docker.com/engine/logging/)

---

## 5.2. Quy trình debug container theo kiểu SRE

Mục tiêu không phải chạy càng nhiều lệnh càng tốt; mục tiêu là thu hẹp lỗi theo tầng.

### Bước 1 — Xác định timeline và phạm vi

- Bắt đầu lúc nào?
- Một container, một host hay toàn bộ service?
- Có deploy/config/migration nào ngay trước đó?
- Người dùng thấy timeout, 5xx, connection refused hay dữ liệu sai?

### Bước 2 — Trạng thái container

```bash
docker ps -a
docker inspect <container>
```

Kiểm tra status, exit code, error, restart count, `OOMKilled`, health, command và image digest.

### Bước 3 — Log và process

```bash
docker logs --since 15m --timestamps <container>
docker top <container>
```

Xác định process chính có chạy, có crash loop, lỗi permission/config hay không.

### Bước 4 — Runtime configuration

Từ `docker inspect`, kiểm tra:

- Env và command/entrypoint.
- Port bindings.
- Mount source/destination và read-only.
- Network attachment, DNS aliases.
- Resource limits và restart policy.

Không paste toàn bộ `inspect` lên chat/ticket công khai nếu có secret.

### Bước 5 — Network path

Kiểm tra tuần tự:

1. App có listen đúng port và `0.0.0.0` không?
2. DNS name dependency resolve không?
3. TCP connection tới dependency được không?
4. Host port publish đúng không?
5. Firewall/security group/reverse proxy có đúng không?

### Bước 6 — Resource và host

```bash
docker stats --no-stream
docker system df
docker events --since 15m
```

Kiểm tra CPU throttling, memory/OOM, disk space, inode, file descriptor và daemon events.

### Bước 7 — Khắc phục có thể tái lập

- Thay đổi code/Dockerfile/config trong version control.
- Build/tag image mới.
- Deploy/recreate có kiểm soát.
- Xác minh metric/log/health.
- Ghi runbook/postmortem nếu đáng kể.

---

# Phần 6 — Đưa vào production

## 6.1. Bảo mật container

### Checklist có giá trị cao

1. Base image từ nguồn tin cậy, cập nhật thường xuyên.
2. Dùng multi-stage để bỏ compiler và tool không cần ở runtime.
3. Chạy non-root bằng `USER` cố định.
4. Không dùng `--privileged` nếu không có lý do đã review.
5. Drop Linux capabilities không cần thiết; chỉ add tối thiểu.
6. Root filesystem read-only nếu ứng dụng hỗ trợ.
7. Mount chỉ đường dẫn cần thiết; ưu tiên read-only cho config.
8. Không bake secret vào image, layer, build arg hoặc source.
9. Scan dependency/image; tạo SBOM và quản lý ngoại lệ CVE.
10. Bảo vệ Docker socket và Docker daemon API.

### Tại sao Docker socket nguy hiểm?

Một process có quyền điều khiển Docker daemon thường có thể tạo container privileged, mount filesystem host và đạt quyền rất cao trên host. Vì vậy không bind mount `/var/run/docker.sock` tùy tiện.

### Root trong container

Root trong container không luôn tương đương root host, nhưng làm blast radius lớn hơn nếu có misconfiguration hoặc kernel/runtime vulnerability. Non-root là defense-in-depth, không phải toàn bộ security model.

### Image tag và cập nhật

- Tag có thể thay đổi; digest là content-addressed reference.
- Pin quá chặt mà không có automation cập nhật sẽ giữ dependency cũ.
- Giải pháp thực tế: pin để tái lập, đồng thời có bot/pipeline rebuild, scan và promote bản cập nhật.

Nguồn: [Docker Engine security](https://docs.docker.com/engine/security/), [Building best practices](https://docs.docker.com/build/building/best-practices/)

---

## 6.2. Registry, tagging và tính bất biến

### Tagging đề xuất

```text
registry.example.com/team/my-api:1.4.2
registry.example.com/team/my-api:git-a1b2c3d
registry.example.com/team/my-api@sha256:...
```

- SemVer giúp con người hiểu release.
- Git SHA nối image về source commit.
- Digest xác định chính xác nội dung image.

### Quy trình promotion

```text
Build một lần
 -> test/scan
 -> push registry
 -> deploy staging bằng digest
 -> promote cùng digest sang production
```

Không build lại ở mỗi môi trường vì kết quả có thể khác do dependency/tag/cache thay đổi.

### Rollback

Rollback nên trỏ về digest đã biết tốt. Trước khi deploy cần lưu:

- Source commit.
- Image digest.
- Config version.
- Database migration compatibility.
- Người/phê duyệt và thời gian.

---

## 6.3. CI/CD cho Docker image

Một pipeline tối thiểu:

1. Lint và unit test.
2. `docker build --check .`.
3. Build bằng BuildKit, dùng cache có kiểm soát.
4. Chạy integration test trên image vừa build.
5. Scan vulnerability và policy check.
6. Tạo SBOM/provenance nếu tổ chức yêu cầu.
7. Push tag bất biến và ghi nhận digest.
8. Deploy staging, smoke test.
9. Promote đúng digest sang production.
10. Quan sát SLO/error rate/latency và rollback khi vượt guardrail.

### Câu hỏi review pipeline

- Có leak secret qua build log hoặc layer không?
- Cache có thể làm dùng dependency stale không?
- Test chạy trên chính image sẽ deploy hay chỉ chạy trên source?
- Có concurrent build ghi đè mutable tag không?
- Có thể truy vết digest về commit và build run không?
- Rollback có tương thích database migration không?

Nguồn: [Docker CI guides](https://docs.docker.com/build/ci/)

---

## 6.4. Các anti-pattern cần tránh

| Anti-pattern | Vì sao nguy hiểm | Cách tốt hơn |
|---|---|---|
| Deploy `latest` | Không biết nội dung thật | Tag bất biến + pin digest |
| Chạy root/privileged | Blast radius lớn | Non-root, drop capabilities |
| Lưu DB trong writable layer | Recreate mất dữ liệu | Named volume + backup |
| Không resource limit | Noisy neighbor, host instability | Limit + alert + load test |
| Sửa file bằng `docker exec` | Không audit/tái lập | Sửa Git, build và redeploy |
| Log chỉ ở local | Container mất là mất evidence | Central logging |
| Mount Docker socket | Quyền điều khiển host | API/proxy tối thiểu hoặc thiết kế khác |
| Một container nhiều daemon | Khó health/scale/debug | Một concern chính mỗi container |
| Secret trong image/.env commit | Tồn tại trong layer/history/Git | Secrets manager/runtime injection |
| Hard-code container IP | IP đổi khi recreate | User-defined network + DNS name |

---

## 6.5. Bốn bài lab thực hành

### Lab 1 — Nginx container

Mục tiêu:

- Pull và chạy Nginx.
- Publish port 8080.
- Xem logs, inspect, stats.
- Stop, start, remove.

Definition of Done:

- Truy cập được `http://localhost:8080`.
- Giải thích được `8080:80`.
- Biết khác nhau giữa image và container.

### Lab 2 — Đóng gói Python API

Mục tiêu:

- Viết Dockerfile.
- Thêm `.dockerignore`.
- Sắp xếp layer để tận dụng cache.
- Chạy non-root.
- Build, tag và push registry.

Definition of Done:

- Rebuild nhanh khi chỉ sửa source.
- Container chạy với UID không phải 0.
- Image có version rõ ràng.

### Lab 3 — API + PostgreSQL bằng Compose

Mục tiêu:

- Hai service giao tiếp bằng service name.
- Dùng named volume cho PostgreSQL.
- Dùng env file, không commit secret.
- Thêm healthcheck.
- `docker compose config` trước khi `up`.

Definition of Done:

- Xóa/recreate container nhưng dữ liệu còn.
- DB không cần publish ra host.
- API retry khi DB chưa sẵn sàng.

### Lab 4 — GameDay xử lý sự cố

Chủ động tạo và xử lý lần lượt:

1. Sai port.
2. Thiếu environment variable.
3. Sai DNS/service name.
4. Mount sai quyền.
5. Memory limit quá thấp dẫn đến OOM.
6. Deploy tag mới rồi rollback về digest cũ.

Với mỗi lỗi, ghi:

```text
Timeline:
Triệu chứng:
Evidence:
Giả thuyết:
Lệnh kiểm chứng:
Root cause:
Khắc phục:
Phòng ngừa:
```

---

## 6.6. Checklist review trước production

### Image và build

- [ ] Base image tin cậy và có quy trình cập nhật.
- [ ] Multi-stage nếu build tools không cần ở runtime.
- [ ] `.dockerignore` loại secret, Git metadata và dependency local.
- [ ] Build có thể tái lập; image được tag và lưu digest.
- [ ] Scan/SBOM/policy theo chuẩn tổ chức.

### Runtime và security

- [ ] Chạy non-root; không privileged nếu không có review.
- [ ] Chỉ publish port cần thiết.
- [ ] Config và secret không nằm trong image.
- [ ] Root filesystem/read-only mount áp dụng nơi phù hợp.
- [ ] CPU, memory, PID và ulimit được xác định từ test/đo đạc.

### Data và network

- [ ] Dữ liệu cần persist nằm trên volume/storage phù hợp.
- [ ] Có backup, retention và restore test.
- [ ] Service discovery dùng DNS name, không hard-code IP.
- [ ] Database/internal service không lộ ra ngoài không cần thiết.

### Reliability và operations

- [ ] Healthcheck đúng mục đích, timeout hợp lý.
- [ ] App xử lý SIGTERM và shutdown graceful.
- [ ] Logs có cấu trúc, timestamp, request/trace ID; không leak secret.
- [ ] Metrics/alerts/SLO guardrail đã có.
- [ ] Runbook debug và rollback dùng digest đã được thử.

---

# Phụ lục

## A. Cheat sheet ngắn

```bash
# Images
docker image ls
docker build -t app:1.0.0 .
docker pull nginx:alpine
docker push registry.example.com/app:1.0.0
docker image inspect app:1.0.0
docker history app:1.0.0

# Containers
docker run -d --name app -p 8080:8000 app:1.0.0
docker ps -a
docker logs -f --since 10m app
docker inspect app
docker stats --no-stream
docker exec -it app sh
docker stop app
docker rm app

# Volumes
docker volume ls
docker volume inspect pgdata

# Networks
docker network ls
docker network inspect app-net

# Compose
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f api
docker compose exec api sh
docker compose down

# Host usage
docker system df
docker events --since 10m
```

> Cẩn thận với các lệnh `prune`, `rm`, `down -v`: xác minh đúng target và hiểu khả năng mất dữ liệu trước khi chạy.

---

## B. Câu hỏi tự kiểm tra

1. Vì sao container dừng khi PID 1 kết thúc?
2. Image và container khác nhau thế nào?
3. Vì sao `COPY . .` quá sớm làm build chậm?
4. `EXPOSE` khác `-p` thế nào?
5. Khi nào dùng named volume, bind mount và tmpfs?
6. Vì sao container không nên gọi container khác bằng IP?
7. `depends_on` có đảm bảo DB đã sẵn sàng không?
8. Làm sao biết container bị OOM kill?
9. Vì sao không nên chạy root hoặc mount Docker socket?
10. Tại sao deploy bằng digest giúp rollback/audit tốt hơn?
11. Khi container “running” nhưng người dùng nhận 502, bạn debug theo thứ tự nào?
12. Vì sao cùng một image có thể chạy khác nhau giữa staging và production?

Nếu bạn trả lời được các câu trên và hoàn thành bốn lab, bạn đã có nền tảng Docker đủ tốt để chuyển sang Kubernetes mà không học theo kiểu thuộc lệnh.

---

## C. Tài liệu chính thức nên lưu lại

- [Docker Get Started](https://docs.docker.com/get-started/)
- [Dockerfile overview](https://docs.docker.com/build/concepts/dockerfile/)
- [Building best practices](https://docs.docker.com/build/building/best-practices/)
- [Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Networking overview](https://docs.docker.com/engine/network/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)
- [Docker Engine security](https://docs.docker.com/engine/security/)
- [Docker CLI cheat sheet (PDF)](https://docs.docker.com/get-started/docker_cheatsheet.pdf)
