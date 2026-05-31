---
name: matlab-agentic-toolkit
description: Connect AI agents to MATLAB with MCP tools and curated skills for engineering and scientific workflows
triggers:
  - set up MATLAB for AI agents
  - configure MATLAB MCP server
  - install MATLAB agentic toolkit
  - connect agent to MATLAB
  - use MATLAB with AI coding assistant
  - run MATLAB code through agent
  - configure MATLAB skills for agent
  - troubleshoot MATLAB MCP connection
---

# MATLAB Agentic Toolkit

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

The MATLAB Agentic Toolkit enables AI coding agents to work with MATLAB by providing the Model Context Protocol (MCP) server and curated skills. This toolkit gives agents the knowledge and context to write idiomatic MATLAB code, run tests, diagnose errors, and follow best practices without hallucinating functions or missing features.

## What It Does

- **MCP Server Integration**: Automatically installs and configures the MATLAB MCP Core Server to connect your AI agent to MATLAB
- **Curated Skills**: Provides domain-specific expertise across automotive, signal processing, computer vision, robotics, and more
- **Code Execution**: Run MATLAB code, execute tests, perform static analysis, and build apps through MCP tools
- **Best Practices**: Guides agents through MATLAB workflows, conventions, and idiomatic patterns

## Installation

### Prerequisites

- MATLAB R2020b or later
- Git
- AI coding agent (Claude Code, GitHub Copilot, OpenAI Codex, Gemini CLI, or Sourcegraph Amp)

### Basic Installation

Clone the repository:

```bash
git clone https://github.com/matlab/matlab-agentic-toolkit.git
cd matlab-agentic-toolkit
```

Ask your AI agent to set up the toolkit:

```
Set up the MATLAB Agentic Toolkit
```

The agent will:
1. Locate your MATLAB installation(s)
2. Download the MATLAB MCP Core Server
3. Write global agent configuration
4. Register all skills

Start a new session after setup completes.

### Skills-Only Installation (Claude Code)

If you already have the MATLAB MCP Core Server installed:

```bash
claude plugin marketplace add "https://github.com/matlab/matlab-agentic-toolkit"
claude plugin install matlab-core@matlab-agentic-toolkit
```

This adds skills without modifying your existing MCP configuration.

### Alternative MATLAB-Based Installer

For users who want both MATLAB and Simulink toolkits or need advanced configuration:

1. Download `agenticToolkitInstaller.mltbx` from the [Simulink Agentic Toolkit releases](https://github.com/matlab/simulink-agentic-toolkit/releases)
2. Open the file in MATLAB to install
3. Run in MATLAB:

```matlab
setupAgenticToolkit
```

This installer supports:
- Installing both MATLAB and Simulink toolkits
- Connecting to existing MATLAB sessions (`--matlab-session-mode=existing`)
- Project-specific configuration (not just global)
- Zero token consumption during setup

## MCP Tools

After installation, your agent can use these MCP tools to interact with MATLAB:

### evaluate_matlab_code

Execute MATLAB code and return command window output:

```
Run this MATLAB code: x = linspace(0, 2*pi, 100); y = sin(x); plot(x, y);
```

The agent will use `evaluate_matlab_code` to execute and return results.

### run_matlab_file

Execute a MATLAB script or function file:

```
Run the file analyze_data.m with the current data
```

### run_matlab_test_file

Execute MATLAB tests via `runtests` with structured results:

```
Run tests in test_signal_processing.m and show the results
```

### check_matlab_code

Perform static analysis using Code Analyzer:

```
Check this MATLAB code for issues: function out = calc(x) out = x^2 end
```

### detect_matlab_toolboxes

List installed MATLAB version and available toolboxes:

```
What version of MATLAB is running? List the installed toolboxes.
```

## MCP Resources

The server provides these resources for context:

- **matlab_coding_guidelines**: MATLAB coding standards and best practices
- **plain_text_live_code_guidelines**: Format rules for MATLAB Live Scripts

## Available Skills

Skills provide domain-specific expertise to your agent. The toolkit includes:

### Core Skills

**MATLAB Core** — Foundational MATLAB programming, syntax, and workflows
- Array operations and indexing
- Function definitions and anonymous functions
- Control flow and error handling
- Workspace management

**MATLAB Data Import and Analysis** — Data loading, preprocessing, and analysis
- Reading from files (CSV, Excel, JSON, images)
- Data cleaning and transformation
- Statistical analysis and visualization
- Table and timetable operations

**MATLAB Software Development** — Professional development practices
- Object-oriented programming
- Unit testing with matlab.unittest
- Code organization and packaging
- Version control integration

**MATLAB App Building** — Creating interactive applications
- App Designer workflows
- UI components and callbacks
- Deployment options

### Domain-Specific Skills

**Automotive** — Automotive engineering workflows
- CAN/LIN communication
- Vehicle modeling
- Calibration and testing

**Computational Biology** — Bioinformatics and systems biology
- Sequence analysis
- Pathway modeling
- Microarray data analysis

**Image Processing and Computer Vision** — Image analysis and vision algorithms
- Image filtering and enhancement
- Object detection and tracking
- Camera calibration
- Deep learning for vision

**Robotics and Autonomous Systems** — Robotics development
- ROS integration
- Path planning
- Sensor fusion
- Simulation with Gazebo

**Signal Processing** — Digital signal analysis
- Filter design
- Spectral analysis
- Wavelet transforms
- Time-frequency analysis

**Wireless Communications** — Communication system design
- Modulation and coding
- MIMO systems
- 5G NR workflows
- Channel modeling

**RF and Mixed Signal** — RF circuit design and analysis
- S-parameter analysis
- Amplifier design
- PLL design

**Test and Measurement** — Instrument control and data acquisition
- Instrument communication (VISA, GPIB)
- Data acquisition workflows
- Automated testing

**Reporting and Database Access** — Report generation and database connectivity
- MATLAB Report Generator
- Database Toolbox workflows
- Custom report templates

## Common Usage Patterns

### Running MATLAB Code

Ask your agent to execute code directly:

```
Create a matrix A with random values and compute its eigenvalues
```

Agent uses `evaluate_matlab_code`:
```matlab
A = rand(5, 5);
eigenvalues = eig(A);
disp(eigenvalues);
```

### Creating and Testing Functions

```
Write a function to calculate the moving average of a signal, 
then create tests for it
```

Agent creates `movingAverage.m`:
```matlab
function y = movingAverage(x, windowSize)
    % MOVINGAVERAGE Compute moving average of input signal
    %   y = movingAverage(x, windowSize) returns the moving average
    %   of x using a window of size windowSize
    
    arguments
        x (:,1) double
        windowSize (1,1) double {mustBePositive, mustBeInteger}
    end
    
    b = ones(1, windowSize) / windowSize;
    y = filter(b, 1, x);
end
```

And test file `test_movingAverage.m`:
```matlab
classdef test_movingAverage < matlab.unittest.TestCase
    
    methods(Test)
        function testBasicOperation(testCase)
            x = [1 2 3 4 5]';
            y = movingAverage(x, 3);
            testCase.verifyEqual(length(y), length(x));
        end
        
        function testConstantSignal(testCase)
            x = ones(10, 1) * 5;
            y = movingAverage(x, 3);
            % After initial transient, output should equal input
            testCase.verifyEqual(y(3:end), x(3:end), 'AbsTol', 1e-10);
        end
    end
end
```

Then runs tests: `run_matlab_test_file('test_movingAverage.m')`

### Data Analysis Workflow

```
Load data.csv, clean it, and create a visualization showing trends over time
```

Agent creates complete workflow:
```matlab
% Import data
data = readtable('data.csv');

% Clean data - remove rows with missing values
cleanData = rmmissing(data);

% Convert to timetable if date column exists
if ismember('Date', cleanData.Properties.VariableNames)
    cleanData.Date = datetime(cleanData.Date);
    tt = table2timetable(cleanData);
    
    % Plot time series
    figure;
    stackedplot(tt);
    title('Data Trends Over Time');
else
    % Create standard plot
    figure;
    plot(cleanData.Variables);
    legend(cleanData.Properties.VariableNames);
    title('Data Trends');
end
```

### Building a Simple App

```
Create an app that allows users to upload an image and apply filters
```

Agent creates App Designer code or programmatic UI:
```matlab
function imageFilterApp
    % Create figure and components
    fig = uifigure('Name', 'Image Filter App');
    g = uigridlayout(fig, [3 2]);
    
    % Load button
    btnLoad = uibutton(g, 'Text', 'Load Image', ...
        'ButtonPushedFcn', @loadImage);
    btnLoad.Layout.Row = 1;
    btnLoad.Layout.Column = 1;
    
    % Filter dropdown
    dd = uidropdown(g, 'Items', {'None', 'Gaussian', 'Median', 'Edge'});
    dd.Layout.Row = 1;
    dd.Layout.Column = 2;
    dd.ValueChangedFcn = @applyFilter;
    
    % Image axes
    ax = uiaxes(g);
    ax.Layout.Row = [2 3];
    ax.Layout.Column = [1 2];
    
    img = [];
    
    function loadImage(~, ~)
        [file, path] = uigetfile({'*.jpg;*.png;*.bmp', 'Image Files'});
        if file
            img = imread(fullfile(path, file));
            imshow(img, 'Parent', ax);
        end
    end
    
    function applyFilter(~, ~)
        if isempty(img)
            return;
        end
        
        filtered = img;
        switch dd.Value
            case 'Gaussian'
                filtered = imgaussfilt(img, 2);
            case 'Median'
                filtered = medfilt3(img);
            case 'Edge'
                if size(img, 3) == 3
                    gray = rgb2gray(img);
                else
                    gray = img;
                end
                filtered = edge(gray, 'Canny');
        end
        
        imshow(filtered, 'Parent', ax);
    end
end
```

## Configuration

### Global vs. Project Configuration

By default, the toolkit configures your agent globally. To use project-specific configuration with the MATLAB-based installer:

```matlab
setupAgenticToolkit('Scope', 'project')
```

### Connecting to Existing MATLAB Session

To connect the MCP server to an already-running MATLAB session:

```matlab
setupAgenticToolkit('SessionMode', 'existing')
```

Or manually configure the MCP server with `--matlab-session-mode=existing` flag.

### Verifying Installation

Ask your agent:
```
What version of MATLAB is running? List the installed toolboxes.
```

Expected output shows MATLAB version and all installed toolboxes.

## Troubleshooting

### MCP Server Not Responding

1. Check MATLAB is installed and accessible:
```bash
matlab -batch "disp(version)"
```

2. Verify MCP server configuration in your agent's config file
3. Restart your agent session
4. Check MCP server logs (location varies by agent)

### Skills Not Available

If skills aren't loading:

1. Verify installation completed successfully
2. For Claude Code, check plugins:
```bash
claude plugin list
```

3. Reinstall skills-only if needed
4. Start a new agent session

### Code Execution Errors

When `evaluate_matlab_code` fails:

1. Ask agent to use `check_matlab_code` first for syntax validation
2. Verify MATLAB path includes required toolboxes
3. Check for workspace variable conflicts
4. Review MATLAB command window output for detailed errors

### Permission Issues

On Windows, if installation fails with permission errors:
- Run your terminal/agent as administrator
- Ensure MATLAB installation directory is accessible

On macOS/Linux:
- Check file permissions: `ls -la ~/.config/` (or agent-specific config location)
- Ensure MATLAB can be executed: `which matlab`

### Toolbox Detection Issues

If `detect_matlab_toolboxes` returns incomplete results:
```matlab
% Run in MATLAB directly to verify
ver
```

Compare output with agent results. Missing toolboxes may need licensing or installation.

## Security Considerations

**Always review tool calls before execution**. The MCP server executes code with full MATLAB privileges:

1. Validate all `evaluate_matlab_code` calls before approval
2. Review file operations (`run_matlab_file`, `save`, `load`)
3. Check network operations (database connections, web requests)
4. Keep humans in the loop for critical operations
5. Use read-only mode when analyzing untrusted code

**Licensing**: MCP servers must comply with MathWorks Software License Agreement and cannot be shared by multiple users. Contact MathWorks for multi-user deployments.

## Environment Variables

The toolkit respects these environment variables:

- `MATLAB_ROOT`: Override MATLAB installation path detection
- `MCP_SERVER_LOG_LEVEL`: Set logging verbosity (debug, info, warn, error)

Reference in configuration:
```bash
export MATLAB_ROOT=/Applications/MATLAB_R2024b.app
```

## Further Resources

- [Configuration and Troubleshooting Guide](https://github.com/matlab/matlab-agentic-toolkit/blob/main/Configuration_and_Troubleshooting.md)
- [Skills Catalog](https://github.com/matlab/matlab-agentic-toolkit/tree/main/skills-catalog)
- [MATLAB MCP Core Server](https://github.com/matlab/matlab-mcp-core-server)
- [MCP Specification](https://modelcontextprotocol.io/specification/2025-06-18/server/tools)
- [MathWorks Agentic AI Campaign](https://www.mathworks.com/campaigns/offers/next/agentic-ai-with-matlab.html)
