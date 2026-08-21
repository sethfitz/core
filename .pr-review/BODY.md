## Proposed change

`FanEntity.percentage` for VeSync devices is only populated in `manual` and `normal` modes:

```python
current_level = self.device.state.fan_level
if self.device.state.mode in (VS_FAN_MODE_MANUAL, VS_FAN_MODE_NORMAL):
    ...
return None
```

That is the right behaviour for `percentage` — in `auto` or `sleep` no speed has been set by
the user — but it means that for a purifier left in `auto`, which is how these devices are
normally run, the level the unit has chosen for itself is not available anywhere in Home
Assistant. `pyvesync` reports it in every mode; only this gate hides it. The value is visible in
the integration's own diagnostics output, which is a poor place to watch a number that changes
through the day:

```json
{"mode": "auto", "fan_level": 1, "air_quality_value": 9}
```

This adds a diagnostic sensor for the fan level, so the running speed can be seen and graphed
regardless of mode, and leaves `fan.percentage` semantics untouched.

Levels outside the device's supported range are reported as unknown rather than passed through.
That mirrors the guard `fan.py` already carries, for the reason its comment gives — a device can
report a placeholder such as `-1` when no speed applies. Level `0` is passed through, since the
fan platform also treats `0` as a real value.

The sensor is created for any device whose state exposes `fan_level`, using the same
`rgetattr(...) is not None` idiom the neighbouring `filter-life` description uses, so it appears
on fans and purifiers and nowhere else.

## Type of change

- [ ] Dependency upgrade
- [ ] Bugfix (non-breaking change which fixes an issue)
- [ ] New integration (thank you!)
- [x] New feature (which adds functionality to an existing integration)
- [ ] Deprecation (breaking change to happen in the future)
- [ ] Breaking change (fix/feature causing existing functionality to break)
- [ ] Code quality improvements to existing code or addition of tests

## Additional information

- This PR fixes or closes issue: fixes #
- This PR is related to issue: 
- Link to documentation pull request: 
- Link to developer documentation pull request: 
- Link to frontend pull request: 

## Checklist

- [ ] I understand the code I am submitting and can explain how it works.
- [x] The code change is tested and works locally.
- [x] Local tests pass. **Your PR cannot be merged unless tests pass**
- [x] There is no commented out code in this PR.
- [x] I have followed the [development checklist][dev-checklist]
- [x] I have followed the [perfect PR recommendations][perfect-pr]
- [x] The code has been formatted using Ruff (`ruff format homeassistant tests`)
- [x] Tests have been added to verify that the new code works.
- [x] Any generated code has been carefully reviewed for correctness and compliance with project standards.

If user exposed functionality or configuration variables are added/changed:

- [ ] Documentation added/updated for [www.home-assistant.io][docs-repository]

If the code communicates with devices, web services, or third-party tools:

- [ ] The [manifest file][manifest-docs] has all fields filled out correctly.  
      Updated and included derived files by running: `python3 -m script.hassfest`.
- [ ] New or updated dependencies have been added to `requirements_all.txt`.  
      Updated by running `python3 -m script.gen_requirements_all`.
- [ ] For the updated dependencies a diff between library versions and ideally a link to the changelog/release notes is added to the PR description.

To help with the load of incoming pull requests:

- [ ] I have reviewed two other [open pull requests][prs] in this repository.
