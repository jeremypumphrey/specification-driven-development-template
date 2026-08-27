# sdd_boilerplate
A repo of the current best Specification files used to launch new applications

I like to Plan with Claude Opus 4.8 & High

I like to Implement Plans with GPT-5.6 Sol & High

## Updating specs
After making changes to technical or functional Specifications I like to run:

    Functional specifications in /specs/ have changed. Thoroughly review changes and make a plan to implement

    Before drafting the plan, fully read:
    - .github/copilot-instructions.md
    - AGENTS.md
    - All .md files in specs/functional/ and specs/technical/
    - previous plans under /specs/plans/
 
If you approve of the plan you can tell it to execute the plan as stated.

## New projects
For new projects copy the contents of this repo to a new one. 
Make sure the .git info specific to the new repo isn't overwriten. 
Make sure the .github folder and contents is copied
Update your functional specs to reflect your basic project goals.
In /specs/technical/ comment out the unecessary infrastructure-SPECIFIC.md file(s)

When ready, I like to run this to PLAN:

    Create a thorough implementation plan for the features described in each file matching specs/functional/*.md. Do not implement or write code.

    Before drafting the plan, fully read:
    - .github/copilot-instructions.md
    - AGENTS.md
    - All .md files in specs/functional/ and specs/technical/

    /specs/
    ├── functional/     # Human-authoritative functional requirements
    ├── technical/      # Human-authoritative technical constraints
    ├── derived/        # Inferred requirements traceable to authoritative specs
    └── assumptions/    # Unvalidated assumptions required to proceed

    Save the plan to specs/plans/<feature-name>-plan.md (one per feature, or a single combined plan if features are tightly coupled across shared modules, state, or app components — use your judgment and state which you chose).



 
If you approve of the plan you can tell it to execute the plan as stated.
When ready, I like to run this to IMPLEMENT the PLAN:

    Fully implement the plans found in specs/plans/

    For any ambiguity follow the plan first, and if needed reference the original specs in :
    - .github/copilot-instructions.md
    - AGENTS.md
    - All .md files in specs/functional/ and specs/technical/
