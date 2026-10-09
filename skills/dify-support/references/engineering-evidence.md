# Engineering Evidence

Use when the case suggests a known defect, previous internal discussion, planned redesign, pending fix, or deployment question.

Search through OpenConnector:

- Linear `linear/default`: exact issue, description, comments, relations, status, project/release fields;
- Feishu `feishu/default`: original discussion and dated follow-up visible to the connected identity;
- GitHub `github-mcp`: relevant issue, PR, commit, release, and code evidence;
- Source/SigNoz/runtime evidence: actual deployed ref and customer path.

An issue title or search result snippet is insufficient. Verify the exact product path/version, latest dated comments, related issue/PR, release evidence, deployment evidence, and customer recovery separately.

Keep these states distinct:

1. hypothesis or proposal;
2. accepted engineering issue;
3. implementation in progress;
4. PR merged;
5. release artifact available;
6. deployed to the relevant SaaS runtime;
7. affected customer path retested successfully.

Linear `Resolved` or a merged PR does not prove release, deployment, or customer recovery. A deployment does not prove the reported mechanism caused the incident. State only the highest level directly supported.

Linear/Feishu/GitHub writes require explicit authorization for the exact issue, comment, or update. Preparing a draft never authorizes submission.
