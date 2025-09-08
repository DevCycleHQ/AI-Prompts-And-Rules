# DevCycle Feature Cleanup Prompt

<context>
You are helping to clean up a DevCycle feature that is ready for removal. This process involves several steps to safely transition from dynamic feature flagging to static code while maintaining the desired behavior.

DevCycle features contain variables and variations that control different behaviors. When a feature is "complete", it should serve a single static variation to all users in all environments. Variables can then be removed from the codebase and replaced with static values.

Reference: https://docs.devcycle.com/platform/feature-flags/status-and-lifecycle#completing-a-feature
</context>

<instructions>
Follow these steps in order to safely clean up a DevCycle feature:

<step1>
**Analyze Current Production State**
Determine which variation is currently set for all users in the production environment:
1. List the feature's targeting configuration for the production environment
2. Identify which variation is being served to users in production  
3. Note the specific variable values in that variation

Tools to use:

- `list_feature_targeting` to get current targeting rules for production
- `fetch_feature_variations` to see all available variations
- Look for the production environment (has `type: "production"`)
  </step1>

<step2>
**Complete the Feature**
Update the feature status to "complete" and set the static variation:
1. If in the previous step it is unclear which variation is being served to all users in production, then confirm with the user which variation should be the "release variation" before proceeding, otherwise proceed with the all users variation.
2. Use `update_feature_status` to:
   - Set `status: "complete"`
   - Set `staticVariation` to the key of the chosen variation

⚠️ **Important:** Always confirm before making changes to production features!
</step2>

<step3>
**Identify Associated Variables**
Get the variables that belong to this feature:
1. Use `list_features` to get the full feature details including variables
2. Note each variable's key and its static value from the release variation
</step3>

<step4>
**Ask About Code Cleanup**
Ask the user: "Would you like to remove the Variables associated with this feature from the codebase?"

If yes, proceed to Step 5. If no, the cleanup is complete.
</step4>

<step5>
**Remove Variables from Codebase**
For each variable associated with the feature:
1. Identify the variable key and its static value from the release variation
2. Replace or remove any code associated with the variable key
3. Remove any conditional logic based on the variable and replace it with the appropriate static behavior
4. Remove any imports or references that are no longer needed

<code_replacement_patterns>
**Before:** `if (devcycleClient.variable("show-new-ui", false)) { /* new UI */ } else { /* old UI */ }`
**After (if static value is true):** `/* new UI code only */`
**After (if static value is false):** `/* old UI code only */`
</code_replacement_patterns>
</step5>

<step6>
**Clean Up Variable Definitions (Optional)**
After removing variables from code:
1. Consider archiving the variables using `update_variable_status` with `status: "archived"`
2. Or remove variables from the feature if they're no longer needed
</step6>
</instructions>

<safety_checklist>
Before proceeding with any cleanup, verify:

- [ ] Confirmed the correct production variation and its variable values
- [ ] Verified this is indeed the intended behavior for all users
- [ ] Confirmed the feature status change with the user
- [ ] Tested that the static values maintain the desired application behavior
- [ ] Ensured no other features or code depend on these variables
      </safety_checklist>

<post_cleanup_verification>
After completing the cleanup:

1. Verify the feature shows as "Complete" in the DevCycle dashboard
2. Test the application to ensure behavior is as expected
3. Confirm variables are no longer referenced in the codebase
4. Consider archiving the feature itself if no longer needed
   </post_cleanup_verification>

<warning>
This is a potentially destructive operation that affects production behavior. Always proceed with caution and get appropriate approvals before making changes.
</warning>
