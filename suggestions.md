# Workspace Architectural Backlog & Suggestions

Cross-repository registry for multi-repo architectural blueprints, shared distributed systems design, tech debt, and cross-cutting capabilities spanning `karigarji_server`, `karigarji_customer`, and `karigarji_admin_panel`.

---

## Registry

| ID | Repositories Involved | Domain / Capability | Architectural Blueprint & Rationale | Priority | Status | Originating Context | Date |
|---|---|---|---|---|---|---|---|
| **SUG-WORKSPACE-001** | `karigarji_server`, `karigarji_customer`, `karigarji_admin_panel` | Media & Verification Asset Pipeline | **Unified Zero-Egress Media Lifecycle**: <br>1. **Mobile (`karigarji_customer`)**: On-device hardware-accelerated image (WebP) and video (H.264 @ 720p/1080p, 2Mbps) compression using `react-native-compressor`, client SHA-256 generation, and direct binary streaming to pre-signed URLs to prevent network stalls and memory crashes.<br>2. **Backend (`karigarji_server`)**: S3-compatible pre-signed upload orchestration (Cloudflare R2 default for $0 egress fees), metadata tracking, secure access control via short-lived signed read URLs, and tamper-proof verification records.<br>3. **Admin Panel (`karigarji_admin_panel`)**: Media inspection viewer, dispute evidence timeline, and cached signed URL streaming for operations staff. | High | In Progress | Asset management & verification architecture | 2026-10-01 |

---

## Detailed Architectural Specifications

### SUG-WORKSPACE-001: Distributed Media & Verification Asset Pipeline

#### 1. Problem Statement & Motivation
* **Device Registration & Transit Proof**: High-resolution camera captures (6-10MB photos) create significant upload latency on cellular networks and bloat cloud storage.
* **Workmanship Proof Videos**: Uncompressed smartphone recordings (1080p/4K @ 60fps) range from 150MB to 400MB+. Direct uploads choke mobile connections, crash JavaScript runtimes if loaded into RAM, saturate backend application servers if proxied, and incur catastrophic bandwidth/egress costs on AWS S3 ($0.09/GB).
* **Audit & Legal Integrity**: Claims regarding pre-existing device damage, courier handoffs, and technician repair quality require tamper-evident records (SHA-256 hashes, uploader actor tagging, and verified review timestamps).

#### 2. Cross-Repository Architectural Breakdown

```
+-----------------------------------------------------------------------------------+
|                           karigarji_customer (Expo / RN)                         |
|  - Camera capture / photo picker                                                  |
|  - On-device compression via react-native-compressor (WebP / H.264 @ 720p/1080p)  |
|  - Client SHA-256 hash calculation                                                |
|  - Direct HTTP PUT to Presigned Cloudflare R2 / S3 URL                            |
+----------------------------------------+------------------------------------------+
                                         |
                       1. Request URL    | 3. Confirm Upload
                       & 2. Return URL   | with Metadata
                                         v
+-----------------------------------------------------------------------------------+
|                           karigarji_server (NestJS Backend)                       |
|  - S3 / Cloudflare R2 presigned URL generation (PutObjectCommand)                |
|  - Compulsory uploader actor tagging (ActorType + ActorId, zero defaults)         |
|  - MediaAsset database persistence via Prisma                                     |
|  - Short-lived signed GET URLs (GetObjectCommand) for private asset access        |
+----------------------------------------+------------------------------------------+
                                         ^
                                         | 4. Fetch Order Assets & Signed URLs
                                         |
+-----------------------------------------------------------------------------------+
|                        karigarji_admin_panel (React Admin Web)                    |
|  - Verification & dispute resolution inspection workspace                         |
|  - Responsive image viewer with zoom for serial labels & damage closeups          |
|  - Streamed workmanship video player with verification approval / rejection audit |
+-----------------------------------------------------------------------------------+
```
