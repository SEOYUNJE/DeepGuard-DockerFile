# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo holds the Dockerfiles and container-provisioning assets for the DeepGuard project, a deepfake detection system. Each top-level directory corresponds to one service/container:

- `deepguard-mysql/` — MySQL 8.0 database (implemented: Dockerfile + init SQL)
- `deepguard-fastapi/` — FastAPI backend (placeholder, not yet implemented)
- `deepguard-react/` — React frontend (placeholder, not yet implemented)
- `deepguard-celery/` — Celery worker (placeholder, not yet implemented)
- `deepguard-redis/` — Redis (placeholder, not yet implemented)

Only `deepguard-mysql` currently has content. The other directories exist as empty placeholders for future Dockerfiles — do not assume implementation details for them beyond what the README states.

## Commands

Build and run the MySQL container:

```bash
docker build -t deepguard-mysql ./deepguard-mysql
docker run -p 3306:3306 deepguard-mysql
```

Init scripts in `deepguard-mysql/init_sql/` are copied to `/docker-entrypoint-initdb.d/` and run automatically **in filename order** on first container start (they do not re-run on existing volumes). The `NN_` filename prefix controls execution order — when adding a new init script, pick a prefix that reflects correct dependency order (e.g., tables with foreign-key-like references must come after the tables they reference).

## Database schema (deepguard-mysql/init_sql/)

Execution/dependency order, driven by filename prefix:

1. `01_user.sql` — `user` table (id, name, email, hashed_password). Registered users; uploads are allowed without an account (`user_id` is nullable downstream).
2. `02_image_result.sql` — `image_result`: one row per analyzed image, with deepfake `label`/`score`, face-quality metrics (`face_conf`, `face_ratio`, `face_brightness`), and a `status` lifecycle (default `PENDING`).
3. `03_video_result.sql` — `video_result`: one row per analyzed video; same status/label/score/face-quality shape as `image_result`, keyed by `video_loc`.
4. `04_video_meta_result.sql` — `video_meta_result`: one row per video (`video_id` unique), tracking the frame-sampling pipeline funnel: `total_frames` → `num_sampled` → `num_extracted` → `num_detected`.
5. `05_video_frame_result.sql` — `video_frame_result`: one row per sampled frame of a video (`video_id` FK-like, not unique), with per-frame `score`/face-quality metrics and `frame_time`/`frame_index`.

Analysis pipeline model reflected in the schema: an image or video is submitted → status moves through `PENDING` → ... → terminal state, while a video is additionally broken into frames, each scored individually, with a meta row summarizing how many frames survived each stage of the pipeline (sampled → face-extracted → scored).

None of the `*_result` tables declare real foreign keys — `user_id`/`video_id` relationships are enforced only by convention and indexed (`*_idx`) for lookup performance, not integrity.
