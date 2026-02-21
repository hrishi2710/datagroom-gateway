# Upgrade Plan Questions

Please answer the following questions to help refine the upgrade plan. Select one option for each question by replacing `[ ]` with `[x]`.

---

## Question 1: nvm Version Retention

After the upgrade is complete and verified, should Node 12 be kept installed via nvm for future rollback capability?

- [x] **A)** Yes, keep Node 12 installed indefinitely for emergency rollback
- [ ] **B)** Keep Node 12 for 30 days after production deployment, then uninstall
- [ ] **C)** Uninstall Node 12 immediately after successful production deployment
- [ ] **D)** Not sure / Decide after deployment

**Your Answer:** `___`

**Impact:** Keeping Node 12 allows quick rollback via `nvm use 12` without reinstallation.

---

## Question 2: Frontend Repository Details

You mentioned there's a separate frontend repository that uses Socket.IO. Can you provide details for coordination?

- [ ] **A)** Frontend repo URL/name: `_______________` (I'll include coordination steps)
- [x] **B)** Frontend team will handle their own upgrade (just need notification)
- [ ] **C)** I will upgrade both repos myself (please provide frontend repo path)
- [ ] **D)** Need to identify the frontend repo first

**Your Answer:** `___`

**Impact:** Determines if upgrade plan should include frontend coordination steps or just notifications.

---

## Question 3: Phase 5 Socket.IO Testing Strategy

For testing Socket.IO v4.x functionality, how should we proceed given frontend dependency?

- [ ] **A)** Upgrade backend first, then coordinate frontend upgrade before testing
- [ ] **B)** Upgrade both simultaneously in a coordinated deployment
- [x] **C)** Skip Socket.IO upgrade for now (stay on v2.x) and handle separately later
- [ ] **D)** Test with a simple Socket.IO test client script (no frontend needed for initial testing)

**Your Answer:** `Does the v2.x of Socket.io is compatible with node 22? If yes, then leave it. If not, then come up with some plan.`

**Impact:** Option C would simplify the Node upgrade but leave Socket.IO on older version. Option D allows independent backend testing.

---

## Approval

Once you have answered all questions:

- [x] **I have reviewed all questions and provided answers**
- [x] **Proceed with refining the upgrade plan based on my answers**

