# Contributing to STM32F407 Robot Controller Board

Thank you for your interest in contributing to the STM32F407 Robot Controller Board project! This document provides guidelines for contributing to both hardware and software aspects of the project.

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [How to Contribute](#how-to-contribute)
4. [Hardware Contributions](#hardware-contributions)
5. [Software Contributions](#software-contributions)
6. [Documentation Contributions](#documentation-contributions)
7. [Submitting Changes](#submitting-changes)
8. [Style Guidelines](#style-guidelines)
9. [Testing](#testing)

---

## Code of Conduct

### Our Standards

- **Be respectful** and inclusive
- **Be collaborative** and constructive
- **Focus on what is best** for the community
- **Show empathy** towards other community members

### Unacceptable Behavior

- Harassment, discrimination, or trolling
- Publishing others' private information
- Spam or off-topic content
- Other conduct inappropriate in a professional setting

---

## Getting Started

### Prerequisites

Before contributing, make sure you have:

- Familiarity with Git and GitHub
- For hardware: KiCad or Altium Designer
- For software: STM32CubeIDE or ARM GCC toolchain
- A STM32F407 board for testing (recommended)

### Setting Up Development Environment

1. **Fork the repository** on GitHub

2. **Clone your fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/stm32-robot-controller-board.git
   cd stm32-robot-controller-board
   ```

3. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/ivander11/stm32-robot-controller-board.git
   ```

4. **Create a branch** for your work:
   ```bash
   git checkout -b feature/your-feature-name
   ```

---

## How to Contribute

### Types of Contributions Welcome

- 🐛 **Bug reports and fixes**
- ✨ **New features**
- 📝 **Documentation improvements**
- 🧪 **Test coverage improvements**
- 🔧 **Hardware design improvements**
- 💡 **Example applications**
- 🌐 **Translations**

### Reporting Bugs

When filing a bug report, please include:

**Hardware Issues**:
- Board revision
- Power supply specifications
- Motor and encoder specifications
- Photos of setup (if applicable)
- Exact error symptoms

**Software Issues**:
- Firmware version
- Development environment (IDE version, OS)
- Steps to reproduce
- Expected vs actual behavior
- Serial output or error messages
- Code snippets (if applicable)

**Bug Report Template**:
```markdown
## Description
Brief description of the issue

## To Reproduce
Steps to reproduce:
1. 
2. 
3. 

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- Board Revision: 
- Firmware Version: 
- IDE/Toolchain: 
- OS: 

## Additional Context
Any other relevant information
```

### Suggesting Enhancements

Enhancement suggestions should include:

- **Use case**: Why is this needed?
- **Proposed solution**: How should it work?
- **Alternatives considered**: What other approaches did you think of?
- **Implementation details**: Technical approach (if you have ideas)

---

## Hardware Contributions

### PCB Design Guidelines

**Design Tools**:
- Primary: KiCad 6.0 or later
- Alternative submissions in Altium accepted but will be converted

**Design Rules**:
- Follow existing design rules (see docs/HARDWARE_SPECS.md)
- Maintain 4-layer stackup for signal integrity
- Include proper ground planes
- Label all components clearly
- Use standard footprints when possible

**What to Include**:
- Schematic files (.sch or .kicad_sch)
- PCB layout files (.kicad_pcb)
- Bill of Materials (CSV or Excel)
- Gerber files for manufacturing
- 3D models (STEP format)
- Assembly drawings
- Updated documentation

**Testing Hardware Changes**:
- Build and test prototype
- Verify all specifications
- Document any issues found
- Include test results in PR

### Submitting Hardware Changes

1. Create detailed schematics
2. Route PCB following design guidelines
3. Generate manufacturing files (Gerbers, drill files, BOM)
4. Test prototype if possible
5. Document changes thoroughly
6. Submit pull request with:
   - Before/after comparison
   - Test results
   - Photos of prototype (if available)

---

## Software Contributions

### Firmware Development

**Code Organization**:
```
firmware/
├── Core/           # HAL and system files
├── App/            # Application code
│   ├── motor/      # Motor control
│   ├── encoder/    # Encoder handling
│   ├── comm/       # Communication protocols
│   └── utils/      # Utility functions
└── Tests/          # Unit tests
```

**Coding Standards**:

**Style**:
- Use K&R style bracing
- 4 spaces for indentation (no tabs)
- Max line length: 100 characters
- Use meaningful variable names

**Naming Conventions**:
```c
// Functions: CamelCase with module prefix
void Motor_SetSpeed(uint8_t channel, int8_t speed);

// Variables: snake_case
int32_t encoder_position;
uint16_t motor_current;

// Constants: UPPER_SNAKE_CASE
#define MAX_MOTOR_SPEED 100
#define PWM_FREQUENCY 20000

// Structures: CamelCase with _t suffix
typedef struct {
    uint8_t channel;
    int8_t speed;
} MotorConfig_t;
```

**Comments**:
```c
/**
 * @brief Set motor speed
 * @param channel Motor channel (1-6)
 * @param speed Speed value (-100 to +100)
 * @return 0 on success, error code on failure
 */
int Motor_SetSpeed(uint8_t channel, int8_t speed) {
    // Input validation
    if (channel < 1 || channel > 6) {
        return -1;  // Invalid channel
    }
    
    // Implementation
    // ...
}
```

**Error Handling**:
```c
// Use return codes for error handling
#define MOTOR_OK            0
#define MOTOR_ERR_INVALID  -1
#define MOTOR_ERR_DISABLED -2
#define MOTOR_ERR_FAULT    -3

// Check return values
int result = Motor_SetSpeed(1, 50);
if (result != MOTOR_OK) {
    // Handle error
}
```

### Adding New Features

**Before starting**:
1. Check if feature already exists or is in development
2. Discuss in an issue to get feedback
3. Ensure it aligns with project goals

**Development process**:
1. Create feature branch
2. Implement feature following coding standards
3. Add unit tests
4. Update documentation
5. Test on real hardware
6. Submit pull request

### Protocol Changes

Changes to communication protocols require:

- **Backward compatibility** when possible
- **Version negotiation** for breaking changes
- **Updated documentation** in API_REFERENCE.md
- **Example code** demonstrating usage
- **Migration guide** if breaking changes

---

## Documentation Contributions

### Documentation Structure

```
docs/
├── HARDWARE_SPECS.md   # Hardware specifications
├── SOFTWARE_SETUP.md   # Software development guide
├── API_REFERENCE.md    # Communication protocols
├── QUICKSTART.md       # Quick start guide
└── examples/           # Example applications
```

### Documentation Guidelines

**Writing Style**:
- Clear and concise
- Use active voice
- Include examples
- Add diagrams where helpful
- Test all commands/code before documenting

**Formatting**:
- Use Markdown
- Follow existing structure
- Include table of contents for long documents
- Use code blocks with language specifiers
- Add links to related documentation

**Images and Diagrams**:
- Use PNG or SVG format
- Place in `docs/images/` directory
- Optimize file size
- Include alt text for accessibility
- Use consistent styling

---

## Submitting Changes

### Pull Request Process

1. **Update your fork**:
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

2. **Commit your changes**:
   ```bash
   git add .
   git commit -m "Add feature: description"
   ```

3. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

4. **Create Pull Request** on GitHub:
   - Use descriptive title
   - Fill out PR template
   - Link related issues
   - Request review from maintainers

### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update
- [ ] Hardware improvement

## Testing
Describe testing performed:
- [ ] Built successfully
- [ ] Tested on hardware
- [ ] All tests pass
- [ ] Documentation updated

## Checklist
- [ ] Code follows style guidelines
- [ ] Comments added where needed
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tests added/updated
- [ ] Backwards compatible (or documented)

## Screenshots
If applicable, add screenshots or photos
```

### Review Process

1. **Maintainer review**: Code reviewed for quality and standards
2. **Testing**: Changes tested on hardware when possible
3. **Feedback**: Requested changes discussed
4. **Approval**: PR approved by maintainer(s)
5. **Merge**: Changes merged to main branch

**What reviewers look for**:
- Code quality and style
- Test coverage
- Documentation completeness
- Breaking changes properly handled
- Performance implications
- Security considerations

---

## Style Guidelines

### Git Commit Messages

**Format**:
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting changes
- `refactor`: Code restructuring
- `test`: Test additions/changes
- `chore`: Build/tooling changes

**Examples**:
```
feat(motor): add overcurrent protection

Implement overcurrent detection using ADC monitoring
with configurable threshold per channel.

Closes #123

fix(can): correct message ID handling

CAN message IDs were being truncated to 8 bits.
Now properly handling full 11-bit standard IDs.

docs(api): update UART protocol examples

Add more examples for motor control commands
and clarify error response format.
```

### Code Comments

**When to comment**:
- Complex algorithms
- Non-obvious behavior
- Workarounds for hardware quirks
- API documentation
- TODOs (with issue number)

**When NOT to comment**:
- Obvious code
- Restating what code does
- Outdated information

---

## Testing

### Software Testing

**Unit Tests**:
- Write tests for new functions
- Maintain >80% code coverage
- Use appropriate test framework
- Mock hardware interfaces

**Integration Tests**:
- Test on real hardware when possible
- Verify all motor channels
- Test all communication interfaces
- Check error handling

**Regression Tests**:
- Ensure existing functionality still works
- Run full test suite before submitting PR

### Hardware Testing

**Prototype Testing**:
- Power supply under various loads
- All motor channels functional
- Encoder inputs accurate
- Communication interfaces working
- Thermal performance
- EMI compliance (if possible)

**Documentation**:
- Include test results
- Photos of setup
- Oscilloscope captures (if relevant)
- Thermal images (if available)

---

## Questions?

If you have questions about contributing:

1. Check existing documentation
2. Search closed issues
3. Ask in discussions or community forum
4. Contact maintainers

---

## Recognition

Contributors will be:
- Listed in CONTRIBUTORS.md
- Credited in release notes
- Acknowledged in documentation

Thank you for contributing to make this project better! 🚀

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
