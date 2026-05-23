# Superpowers HRM Test Prompts

Use these prompts to check whether Roo applies the new workflow.

## 1. Flow Research

```text
Dung HRM Flow Researcher trace flow BaoCaoTongHop tu Vue component toi API module, backend/router/service/query neu co. Khong sua file. Neu khong xac minh duoc query key hoac response shape, noi ro can kiem tra them.
```

Expected behavior:

- reads actual files
- reports component, API function, endpoint/query path if present
- does not edit files
- does not guess response shape

## 2. Refactor Planning

```text
Lap plan tach logic lon trong BaoCaoTongHop.vue thanh composable/component nho hon. Truoc khi plan, doc file hien tai va neu can doc API module lien quan. Khong implement.
```

Expected behavior:

- uses planning before edits
- lists files likely affected
- preserves response checks, permissions, worker/report behavior
- gives verification checklist

## 3. Debugging

```text
Man hinh ThongKeHopDong khong load du lieu. Dung workflow debugging: reproduce/trace/root cause truoc khi de xuat fix. Khong sua file cho den khi co root cause.
```

Expected behavior:

- gathers symptom and evidence
- traces component -> API -> backend/query
- compares with working pattern
- proposes minimal fix only after root cause

## 4. TDD Implementation

```text
Them validation backend cho mot endpoint registry da co. Uu tien TDD: viet test fail truoc, verify fail, implement minimal, verify pass. Khong doi schema DB.
```

Expected behavior:

- uses existing pytest style
- avoids DB schema changes
- checks query key and params
- reports exact verification

## 5. Review and Release

```text
Review thay doi vua lam o module ChamCong truoc khi deploy. Tap trung Query Registry, permission, API response shape, worker/report protocol, auth/heartbeat neu lien quan. Findings truoc, summary sau.
```

Expected behavior:

- review findings first by severity
- checks HRM-specific regression risks
- names missing verification
- does not focus on style-only comments
