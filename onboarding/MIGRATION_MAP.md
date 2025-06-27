# BSV Library Onboarding Content Migration Map

This document tracks the migration of every file from the current structure to the optimized structure.

## Migration Status Legend
- 🔄 To be migrated
- ✅ Migrated
- 📝 Needs content update
- 🔀 Split into multiple files
- 🔗 Merged with other content

## Content Migration Map

### 00-welcome/ (NEW SECTION)
- 🔄 `README.md` → Create new universal welcome page
- 🔄 `universal-overview.md` → Create from existing overviews
- 🔄 `quick-wins.md` → Extract from various sources
- 🔄 `navigation-guide.md` → Create new navigation guide

### 01-getting-started/
- 🔄 `01-getting-started/README.md` → `01-getting-started/README.md`
- 🔄 `01-getting-started/examples.md` → `01-getting-started/examples.md`
- 🔄 `01-getting-started/live-demos.md` → `01-getting-started/live-demos.md`
- 🔄 `01-getting-started/metanet-desktop.md` → `01-getting-started/metanet-desktop.md`
- 🔄 `01-getting-started/metanet-mobile.md` → `01-getting-started/metanet-mobile.md`
- 🔄 Create `01-getting-started/wallet-setup.md` from existing content
- 🔄 Create `01-getting-started/first-transaction.md` from existing content
- 🔄 `02-pathways/README.md` → `01-getting-started/choose-your-path.md`

### 02-foundations/
- 🔄 `01-foundations/bsv-evolution.md` → `02-foundations/bsv-evolution.md`
- 🔄 Create `02-foundations/README.md` → New foundations overview
- 🔄 Create `02-foundations/core-concepts.md` → Extract from various sources
- 🔄 Create `02-foundations/why-bsv.md` → Consolidate value propositions
- 🔄 Create `02-foundations/ecosystem-overview.md` → From ecosystem components

### 03-learning-pathways/technical/
- 🔄 `02-pathways/technical/README.md` → `03-learning-pathways/technical/README.md`
- 🔄 `02-pathways/technical/developer-faq.md` → `03-learning-pathways/technical/developer-faq.md`
- 🔄 `02-pathways/technical/live-demos-technical.md` → `03-learning-pathways/technical/live-demos.md`
- 🔄 `02-pathways/technical/01-environment-setup/` → `03-learning-pathways/technical/01-environment-setup/`
- 🔄 `02-pathways/technical/02-wallet-integration/` → `03-learning-pathways/technical/02-wallet-integration/`
- 🔄 `02-pathways/technical/03-building-applications/` → `03-learning-pathways/technical/03-building-apps/`
- 🔄 `02-pathways/technical/04-distributed-architecture/` → `03-learning-pathways/technical/04-distributed-arch/`
- 🔄 `02-pathways/technical/05-production-deployment/` → `03-learning-pathways/technical/05-production/`
- 🔄 `02-pathways/technical/06-advanced-patterns/` → `03-learning-pathways/technical/06-advanced-patterns/`
- 🔄 All Advanced STOs content → `03-learning-pathways/technical/07-advanced-stos/`

### 03-learning-pathways/business/
- 🔄 `02-pathways/business/README.md` → `03-learning-pathways/business/README.md`
- 🔄 `02-pathways/business/bsv-overview.md` → `03-learning-pathways/business/bsv-overview.md`
- 🔄 `02-pathways/business/case-studies.md` → `03-learning-pathways/business/case-studies.md`
- 🔄 `02-pathways/business/implementation-guide.md` → `03-learning-pathways/business/implementation-guide.md`
- 🔄 `02-pathways/business/live-demos-business.md` → `03-learning-pathways/business/live-demos.md`
- 🔄 `02-pathways/business/value-propositions/` → `03-learning-pathways/business/value-propositions/`
- 🔄 `02-pathways/business/05-advanced-business-stos/` → `03-learning-pathways/business/advanced-stos/`

### 03-learning-pathways/enterprise/
- 🔄 `02-pathways/enterprise/README.md` → `03-learning-pathways/enterprise/README.md`
- 🔄 `02-pathways/enterprise/architecture.md` → `03-learning-pathways/enterprise/architecture.md`
- 🔄 `02-pathways/enterprise/deployment-strategies.md` → `03-learning-pathways/enterprise/deployment-strategies.md`
- 🔄 `02-pathways/enterprise/integration-patterns.md` → `03-learning-pathways/enterprise/integration-patterns.md`
- 🔄 `02-pathways/enterprise/governance-risk.md` → `03-learning-pathways/enterprise/governance-risk.md`
- 🔄 `02-pathways/enterprise/regulatory-compliance.md` → `03-learning-pathways/enterprise/regulatory-compliance.md`
- 🔄 `02-pathways/enterprise/security-audit.md` → `03-learning-pathways/enterprise/security-audit.md`
- 🔄 All risk management content → `03-learning-pathways/enterprise/risk-management/`
- 🔄 All case studies → `03-learning-pathways/enterprise/case-studies/`
- 🔄 All implementation content → `03-learning-pathways/enterprise/implementation/`
- 🔄 `02-pathways/enterprise/08-reference/` → `03-learning-pathways/enterprise/reference/`

### 03-learning-pathways/academic/
- 🔄 `02-pathways/academic/README.md` → `03-learning-pathways/academic/README.md`
- 🔄 `02-pathways/academic/web3-and-blockchain/stem-course/` → `03-learning-pathways/academic/stem-course/`
- 🔄 `02-pathways/academic/05-advanced-academic-stos/` → `03-learning-pathways/academic/advanced-stos/`
- 🔗 Merge duplicate academic content from `academic-pathway/` and `advanced-academic-stos/`

### 04-specialized-topics/privacy-identity/
- 🔄 All content from `liz-stuff/` → `04-specialized-topics/privacy-identity/`
- 🔄 Organize into 5 learning paths as documented

### 04-specialized-topics/scrypt-development/
- 🔄 Convert all sCrypt HTML documentation → `04-specialized-topics/scrypt-development/`
- 🔄 `scrypt-documentation/*.md` → Organize into logical sections

### 04-specialized-topics/implementation-strategies/
- 🔄 `04-implementation-strategy/enterprise-risk-transformation.md` → `04-specialized-topics/implementation-strategies/enterprise-risk.md`
- 🔄 `04-implementation-strategy/government-and-regulatory-implementation.md` → `04-specialized-topics/implementation-strategies/government-regulatory.md`
- 🔄 `04-implementation-strategy/sector-specific-applications.md` → `04-specialized-topics/implementation-strategies/sector-specific.md`

### 05-hackathon-pack/
- 🔄 `05-hackathon-essentials/quick-start-guide.md` → `05-hackathon-pack/quick-start-guide.md`
- 🔄 Create new content for expanded hackathon resources

### 06-resources/
- 🔄 `03-resources/README.md` → `06-resources/README.md`
- 🔄 `03-resources/tools-reference.md` → `06-resources/tools-reference.md`
- 🔄 `03-resources/critical-docs.md` → `06-resources/critical-docs.md`
- 🔄 `03-resources/community.md` → `06-resources/community.md`
- 🔄 `03-resources/troubleshooting.md` → `06-resources/troubleshooting.md`
- 🔄 `03-resources/bsv-mcp-comprehensive-guide.md` → `06-resources/bsv-mcp-guide.md`

### 07-assessments/
- 🔄 `assessments/assessment-framework.md` → `07-assessments/assessment-framework.md`
- 🔄 Create pathway-specific assessments

### 08-ecosystem-components/
- 🔄 `BSV_ECOSYSTEM_COMPONENTS.md` → `08-ecosystem-components/bsv-components.md`
- 🔄 `brcs/brc-standards.md` → `08-ecosystem-components/brc-standards.md`
- 🔄 Create network topology content

### 09-reference-library/
- 🔄 All planning documents → `09-reference-library/planning-docs/`
- 🔄 All migration guides → `09-reference-library/migration-guides/`
- 🔄 Technical specifications → `09-reference-library/technical-specs/`

### 10-system-overview/
- 🔄 `00-system-overview/README.md` → `10-system-overview/README.md`
- 🔄 Create additional system documentation

### Files to Archive (not delete)
- `technical-backup-20250611/` → Archive as backup
- `bsv-library/onboarding/` → Archive duplicate nesting
- `packages/onboarding/` → Archive duplicate content
- Empty directories → Remove after migration

## Migration Process

1. **Phase 1**: Create new directory structure
2. **Phase 2**: Copy files according to this map
3. **Phase 3**: Update internal links
4. **Phase 4**: Create new SUMMARY.md
5. **Phase 5**: Validate all content accessible
6. **Phase 6**: Archive old structure

## Notes
- All content will be preserved
- No files will be deleted, only reorganized
- HTML files will be converted to Markdown
- Duplicate content will be merged
- New overview files will be created to improve navigation