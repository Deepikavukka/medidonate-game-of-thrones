# MediDonate

## Current State
- Full-stack medicine donation platform with Motoko backend and React frontend
- `createDonation` works but fails silently -- errors are caught and show only a generic toast message
- `updateStatus` and `batchUpdateStatus` are **admin-only** (guarded by `AccessControl.isAdmin`), but admin initialization depends on a `caffeineAdminToken` URL parameter that users don't have -- meaning status updates silently fail for all users including the first user
- The `DonationModal` submits donations but if the actor is missing or backend rejects, the user only sees "Something went wrong"
- `useActor` calls `_initializeAccessControlWithSecret` with a token from URL params which may be empty, so admin role is never established in practice

## Requested Changes (Diff)

### Add
- Better error messages in DonationModal: show the actual error text in the toast so the user understands what failed

### Modify
- Backend `updateStatus`: allow the **donation owner** OR an admin to update status (currently admin-only). This matches the pattern already used by `updateDonation` and `deleteDonation`.
- Backend `batchUpdateStatus`: keep admin-only (batch operations are a power feature)
- `DonationModal` error catch block: extract and display the real error message from the thrown error
- `StatusUpdateDropdown` error toast: show actual error detail

### Remove
- Nothing

## Implementation Plan
1. **Backend (main.mo)**: Change `updateStatus` to allow caller who is the donation owner OR admin (same pattern as `updateDonation` and `deleteDonation`)
2. **Frontend DonationModal**: In the `catch` block, read `(e as Error).message` and show it in the toast instead of generic text
3. **Frontend StatusUpdateDropdown**: Same fix -- show real error in toast
4. Validate (lint + typecheck + build)
