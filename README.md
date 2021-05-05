# jacoco-badge-generator

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/cicirello/jacoco-badge-generator?label=Marketplace&logo=GitHub)](https://github.com/marketplace/actions/jacoco-badge-generator)
[![build](https://github.com/cicirello/jacoco-badge-generator/actions/workflows/build.yml/badge.svg)](https://github.com/cicirello/jacoco-badge-generator/actions/workflows/build.yml)
[![CodeQL](https://github.com/cicirello/jacoco-badge-generator/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/cicirello/jacoco-badge-generator/actions/workflows/codeql-analysis.yml)
[![License](https://img.shields.io/github/license/cicirello/jacoco-badge-generator)](https://github.com/cicirello/jacoco-badge-generator/blob/main/LICENSE)
![GitHub top language](https://img.shields.io/github/languages/top/cicirello/jacoco-badge-generator)

The jacoco-badge-generator GitHub Action parses a `jacoco.csv` from a JaCoCo coverage report,
computes coverage percentages from [JaCoCo's Instructions and Branches counters](https://www.jacoco.org/jacoco/trunk/doc/counters.html), and 
generates badges for one or both of these (configurable with action inputs) to provide an easy 
to read visual summary of the code coverage of your test cases. The action supports
both the basic case of a single `jacoco.csv`, as well as multi-module projects in which
case the action can produce coverage badges from the combination of the JaCoCo reports
from all modules, provided that the individual reports are independent.

_The developers of the jacoco-badge-generator GitHub Action are not affiliated 
with the developers of JaCoCo, although we are a fan and user of their excellent 
test coverage tool._ 

The documentation is organized into the following sections:
* [The Coverage Metrics](#the-coverage-metrics): Explains the JaCoCo metrics that are supported by the badge generator, such as what they measure, and why they were chosen for inclusion for the jacoco-badge-generator GitHub Action.
* [Badge Style and Content](#badge-style-and-content): Provides examples of the appearance of the badges that are generated, including a description of the color scheme used, and the formatting of the percentages.
* [Inputs](#inputs): Detailed descriptions of the action inputs.
* [Outputs](#outputs): Detailed descriptions of the action inputs.
* [Example Workflows](#example-workflows): Example GitHub workflows demonstrating usage of the jacoco-badge-generator action.
* [Multi-Module Example Workflows](#multi-module-example-workflows): Example GitHub workflows demonstrating usage of the jacoco-badge-generator action with multi-module projects.


## The Coverage Metrics

The jacoco-badge-generator GitHub Action currently supports generating badges for 
the two primary coverage metrics generated by JaCoCo: Instructions (C0 Coverage), and 
Branches (C1 Coverage). Here is a summary of what these compute and why they were chosen
for inclusion by this badge generator.

### Instructions Coverage (C0 Coverage)

The default behavior of the badge generator is to generate only the Instructions Coverage
badge, which is labeled on the badge simply as "coverage". JaCoCo 
measures [C0 Coverage](https://www.jacoco.org/jacoco/trunk/doc/counters.html)
from the Java bytecode instructions in the compiled `.class` files. One of the advantages
to counting the bytecode instructions executed or missed, rather than lines of source code, 
is that it is independent of coding style and formatting. As a simple example, consider
the sequence of Java statements to swap the values in two 
variables: `int temp = a; a = b; b = temp;`.  A line counter will count this as 1 line if
written on a single line, or 3 lines if each statement is written on its own line. However,
JaCoCo's instructions counter treats these two cases as equivalent since they compile
to the same bytecode. Consider a more complex example of calling a method while passing a simple
value, such as `foo(5)` versus passing the result of a calculation to the method, such as
`foo(2.0 + bar/11.0)`. Line counting considers both of these as 1 line; while the second case
will factor in more heavily into JaCoCo's instruction counting than will the first case. For
these reasons, although JaCoCo also provides line coverage data, we do not currently support
generating a badge from JaCoCo's line counter data. JaCoCo's use of bytecode instructions
in its definition of C0 Coverage is a more meaningful measure of coverage than is counting
lines of code.

### Branches Coverage (C1 Coverage)

The badge generator also optionally supports generating a badge for Branches Coverage
(or C1 Coverage), with the generated badge labeled as "branches".  See 
the [inputs](#inputs) section for a description of the action
inputs.  JaCoCo 
measures [C1 Coverage or Branches Coverage](https://www.jacoco.org/jacoco/trunk/doc/counters.html)
from the Java bytecode in the compiled `.class` files, so the result may be a bit
different than what you might expect from branch coverage. At first, you may even mistakenly
guess that it is counting conditions (C2 coverage) rather than branches, but it is counting
branches (in bytecode rather than in source code). Consider this example to illustrate
the difference: `if (a && b) foo(); else bar();`. If we count branches in source code,
there are 2 branches, which would require a minimum of two tests for full coverage, one where
both `a` and `b` are `true`, and a second test where at least one of them is `false`.
If we instead count conditions, there are 4 conditions (`a==true`, `a==false`, `b==true`, 
`b==false`), which can be covered with as few as two tests (e.g., `a==true` and `b==false`
as test one, and `b==true` and `a==false` as test two), without actually covering both
branches. JaCoCo's branches counter is neither of these. JaCoCo's branches counter
counts branches in the Java bytecode. What does that mean for this example? Imagine instead
that the if statement above was written as a pair of nested if 
statements: `if (a) { if (b) { foo(); } else { bar(); } } else { bar(); }`. There are
a total of 4 branches in this case (and is essentially what JaCoCo would count as branches).
To cover all 4 branches would require a minimum of 3 test cases: one where `a` and `b` are both
`true`, one in which `a` is `true` and `b` is `false`, and a third where `a` is `false` and `b`'s 
value doesn't matter. In this way, JaCoCo's branches counter leads to a stronger form
of C1 Coverage than is usually implied by branches coverage.


## Badge Style and Content

The badges that are generated are inspired by the style of the badges 
of [Shields.io](https://github.com/badges/shields), however, the badges are entirely generated
within the jacoco-badge-generator GitHub Action, with no external calls.  Here are
a few samples of what the badges look like:
* ![Coverage 100%](https://github.com/cicirello/jacoco-badge-generator/blob/main/tests/100.svg) We use bright green for 100% coverage.
* ![Coverage 99.9%](https://github.com/cicirello/jacoco-badge-generator/blob/main/tests/999.svg) We use green for coverage from 90% up through 99.9%.
* ![Coverage 80%](https://github.com/cicirello/jacoco-badge-generator/blob/main/tests/80.svg) We use yellow green for coverage from 80% up through 89.9%.
* ![Coverage 70%](https://github.com/cicirello/jacoco-badge-generator/blob/main/tests/70.svg) We use yellow for coverage from 70% up through 79.9%.
* ![Coverage 60%](https://github.com/cicirello/jacoco-badge-generator/blob/main/tests/60.svg) We use orange for coverage from 60% up through 69.9%.
* ![Coverage 59.9%](https://github.com/cicirello/jacoco-badge-generator/blob/main/tests/599.svg) We use red for coverage from 0% up through 59.9%.
* ![Branches Coverage 99.9%](https://github.com/cicirello/jacoco-badge-generator/blob/main/tests/999b.svg) A sample of a branch coverage badge.

The coverage displayed in the badge is the result of truncating to one 
decimal place.  If that decimal place is 0, then it is displayed as an 
integer.  The rationale for truncating to one decimal place, rather than 
rounding is to avoid displaying a just failing coverage as passing. For
example, if the user of the action considers 80% to be a passing level,
then we wish to avoid the case of 79.9999% being rounded to 80% (it will
instead be truncated to 79.9%).  


## Inputs

All inputs include default values, and are thus optional provided the
defaults are relevant to your use-case.

### `jacoco-csv-file`

This input is the full path, relative to the root of the repository, to 
the `jacoco.csv` file, including filename.  It defaults 
to `target/site/jacoco/jacoco.csv`, which is the default location and filename
assuming you are using the JaCoCo Maven plugin and don't change the default
output location. 

If you have a multi-module project, you can pass the paths (including filenames)
to all of the `jacoco.csv` files for all of the sub-projects. Separate these by spaces,
and in particular see the [Multi-Module Example Workflows](#multi-module-example-workflows)
for an example of how to do this. Multi-module support is limited to cases where
each module has its own test coverage report, and where those reports don't overlap.

The action assumes that all reports passed via this input are 
independent of each other. If you are using matrix testing, such that 
each group of tests produces a report, and where the groups overlap in what
they are testing (e.g., one group tests a portion of a class or method, 
and another group tests another portion, etc), then the coverage computed by 
this action will not be correct. The csv reports don't contain enough information
to properly merge such overlapping reports. If this applies to your use-case, then
you will need to have JaCoCo produce a single JaCoCo report first (for example, 
see [jacoco:report-aggregate](https://www.jacoco.org/jacoco/trunk/doc/report-aggregate-mojo.html)). 

### `badges-directory`

This input is the directory for storing badges, relative to the root of the 
repository. The default is `.github/badges`. The action will create the badges
directory if it doesn't already exist, although the action itself does not commit.

### `generate-coverage-badge`

This input controls whether or not to generate the coverage badge (Instructions 
Coverage), and defaults to `true`.

### `coverage-badge-filename`

This input is the filename for the coverage badge (Instructions or C0 
Coverage). The default filename 
is `jacoco.svg`. The file format is an `svg`. The badge file will be 
created within the `badges-directory`
directory. __The action doesn't commit the badge file. You will 
need to have additional steps in your workflow to do that.__

### `generate-branches-badge`

This input controls whether or not to generate the branches coverage badge, and defaults
to `false`. This defaults to `false` to avoid surprising users who upgrade from earlier
versions with a badge they didn't know would be generated.

### `branches-badge-filename`

This input is the filename for the branches coverage badge (C1 
Coverage). The default filename 
is `branches.svg`. The file format is an `svg`. The badge file will be 
created within the `badges-directory`
directory. __The action doesn't commit the badge file. You will 
need to have additional steps in your workflow to do that.__

### `on-missing-report`

This input controls what happens if one or more `jacoco.csv` files do not exist.
This input accepts one of three possible values: `fail`, `quiet`, or `badges`.
The behavior of these is defined as follows:
* The default is `on-missing-report: fail`, in which case the action will 
  return a non-zero exit code (causing the workflow run to fail) if one 
  or more files listed in the `jacoco-csv-file` input do not exist, or if
  an empty list of files is passed to the action. We recommend that you use
  this default since missing coverage report files in most cases probably means
  that there is either a bug in your workflow (e.g., typo in path to jacoco.csv)
  or that something went wrong in an earlier step (e.g., unit tests failed, halting
  generation of the coverage report).
* You can use `on-missing-report: quiet` if you would rather the workflow
  itself not fail, in which case the action will instead quietly exit 
  without producing badges if any JaCoCo reports are missing.
* Although not recommended, a third option, `on-missing-report: badges`, will
  cause the action to produce badges from the report files that do exist, simply
  ignoring missing report files, provided that at least one such report file 
  exists. We do not recommend this option since such a case is likely due to an 
  error in your workflow, and any badges produced are likely computed with missing data.

Regardless of value passed to this input, the action will log warnings for
any files listed in the `jacoco-csv-file` input that do not exist, for your 
inspection in the workflow run. 


## Outputs

The action also outputs the actual computed coverage percentages as double-precision
floating-point numbers. So you can add a step to your workflow to access these if 
desired (these action outputs are values in the interval from 0.0 to 1.0).

### `coverage`

This output is the actual computed coverage percentage in the interval 
from 0.0 to 1.0.  This is coverage computed from the instructions
coverage data in the JaCoCo csv report.

### `branches`

This output is the actual computed branches coverage percentage 
in the interval from 0.0 to 1.0.  This is the percentage of branches
covered, computed from the branches data in the JaCoCo csv report.


## Example Workflows

### Prerequisite: Running JaCoCo

The example workflows assume that you are using Maven to build and test
a Java project, and that you have the `jacoco-maven-plugin`
configured in your `pom.xml` in the test phase with something
along the lines of the following:

```XML
<build>
  <plugins>
    <plugin>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <version>0.8.7</version>
      <executions>
        <execution>
          <goals>
            <goal>prepare-agent</goal>
          </goals>
        </execution>
        <execution>
          <id>generate-code-coverage-report</id>
          <phase>test</phase>
          <goals>
            <goal>report</goal>
          </goals>
        </execution>
      </executions>
      </plugin>
  </plugins>
</build>
```

Note that the jacoco-badge-generator action has been tested with
the `jacoco.csv` files generated by `jacoco-maven-plugin` versions
0.8.6 and 0.8.7, and has not been tested with earlier versions
of JaCoCo.

### Example Workflow 1: Generate instructions (or C0) coverage badge only.

This sample workflow runs on pushes to the main
branch.  It first sets up Java, and runs the tests with Maven. If you
have JaCoCo configured to run during the test phase, this will also
produce the JaCoCo reports.  The jacoco-badge-generator action is then
run to parse the `jacoco.csv`, compute the coverage percentage, and
generate the badge.  The coverage percentage is then logged in the
workflow so you can inspect later if necessary. The next step of the workflow
checks if any changes were made to the badge, and if so, does a commit and a push (note
that there are also GitHub Actions that you can use for that step). And 
finally, the JaCoCo coverage reports are uploaded as a workflow
artifact using the [actions/upload-artifact](https://github.com/actions/upload-artifact)
GitHub Action, so you can inspect them if necessary.

```yml
name: build

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up the Java JDK
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'

    - name: Build with Maven
      run: mvn -B test

    - name: Generate JaCoCo Badge
      id: jacoco
      uses: cicirello/jacoco-badge-generator@v2.1.1

    - name: Log coverage percentage
      run: |
        echo "coverage = ${{ steps.jacoco.outputs.coverage }}"
        echo "branch coverage = ${{ steps.jacoco.outputs.branches }}"

    - name: Commit the badge (if it changed)
      run: |
        if [[ `git status --porcelain` ]]; then
          git config --global user.name 'YOUR NAME HERE'
          git config --global user.email 'YOUR-GITHUB-USERID@users.noreply.github.com'
          git add -A
          git commit -m "Autogenerated JaCoCo coverage badge"
          git push
        fi

    - name: Upload JaCoCo coverage report
      uses: actions/upload-artifact@v2
      with:
        name: jacoco-report
        path: target/site/jacoco/
```

### Example Workflow 2: Generate instructions coverage and branches coverage badges.

This example workflow is just like the above example, however, it generates
both badges (instructions coverage percentage and branches coverage percentage).

```yml
name: build

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up the Java JDK
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'

    - name: Build with Maven
      run: mvn -B test

    - name: Generate JaCoCo Badge
      id: jacoco
      uses: cicirello/jacoco-badge-generator@v2.1.1
      with:
        generate-branches-badge: true

    - name: Log coverage percentage
      run: |
        echo "coverage = ${{ steps.jacoco.outputs.coverage }}"
        echo "branch coverage = ${{ steps.jacoco.outputs.branches }}"

    - name: Commit the badge (if it changed)
      run: |
        if [[ `git status --porcelain` ]]; then
          git config --global user.name 'YOUR NAME HERE'
          git config --global user.email 'YOUR-GITHUB-USERID@users.noreply.github.com'
          git add -A
          git commit -m "Autogenerated JaCoCo coverage badge"
          git push
        fi

    - name: Upload JaCoCo coverage report
      uses: actions/upload-artifact@v2
      with:
        name: jacoco-report
        path: target/site/jacoco/
```

## Multi-Module Example Workflows

### Example Workflow 3: Generate Instructions and Coverage Badges for a Multi-Module Project.

This example workflow generates both badges (instructions coverage percentage 
and branches coverage percentage) for a multi-module project. The badges that are generated
are computed over all modules. To do so, simply pass the paths to all of the JaCoCo reports
that you want to include via the `jacoco-csv-file` input. The `>` is just Yaml's way of writing
a string across multiple lines. You can also just list all on a single space-separated line,
but your workflow file will be easier for you to read if you put them one per line.
In this example, there are three subprojects: `module1`, `module2`, and `module3`.

```yml
name: build

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up the Java JDK
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'

    - name: Build with Maven
      run: mvn -B test

    - name: Generate JaCoCo Badge
      id: jacoco
      uses: cicirello/jacoco-badge-generator@v2.1.1
      with:
        generate-branches-badge: true
        jacoco-csv-file: >
          module1/target/site/jacoco/jacoco.csv
          module2/target/site/jacoco/jacoco.csv
          module3/target/site/jacoco/jacoco.csv

    - name: Log coverage percentage
      run: |
        echo "coverage = ${{ steps.jacoco.outputs.coverage }}"
        echo "branch coverage = ${{ steps.jacoco.outputs.branches }}"

    - name: Commit the badge (if it changed)
      run: |
        if [[ `git status --porcelain` ]]; then
          git config --global user.name 'YOUR NAME HERE'
          git config --global user.email 'YOUR-GITHUB-USERID@users.noreply.github.com'
          git add -A
          git commit -m "Autogenerated JaCoCo coverage badge"
          git push
        fi
```

### Example Workflow 4: Multi-Module Project with Separate Badges for Each Module.

If you would prefer to generate separate coverage badges for each of the
modules of a multi-module project, then just include multiple steps of the
`jacoco-badge-generator` in your workflow, such as in this example. Be sure to use
the inputs to specify names for the badge files, otherwise with the defaults
the subsequent steps will overwrite the previous. This example assumes that there
are two modules.

```yml
name: build

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up the Java JDK
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'

    - name: Build with Maven
      run: mvn -B test

    - name: Generate JaCoCo Badges for Module 1
      id: jacocoMod1
      uses: cicirello/jacoco-badge-generator@v2.1.1
      with:
        generate-branches-badge: true
        jacoco-csv-file: module1/target/site/jacoco/jacoco.csv
        coverage-badge-filename: jacoco1.svg
        branches-badge-filename: branches1.svg

    - name: Generate JaCoCo Badges for Module 2
      id: jacocoMod2
      uses: cicirello/jacoco-badge-generator@v2.1.1
      with:
        generate-branches-badge: true
        jacoco-csv-file: module2/target/site/jacoco/jacoco.csv
        coverage-badge-filename: jacoco2.svg
        branches-badge-filename: branches2.svg

    - name: Commit the badge (if it changed)
      run: |
        if [[ `git status --porcelain` ]]; then
          git config --global user.name 'YOUR NAME HERE'
          git config --global user.email 'YOUR-GITHUB-USERID@users.noreply.github.com'
          git add -A
          git commit -m "Autogenerated JaCoCo coverage badge"
          git push
        fi
```

## Examples in Other Projects

If you would like to see examples where the action is actively used, here 
are a few repositories that are actively using the `jacoco-badge-generator` action.
The table provides a link to repositories using the action, and direct links to the
relevant workflow as well as the Maven `pom.xml` so you can see how JaCoCo is 
configured. Note that in most of these JaCoCo is configured within a Maven profile
within the `pom.xml`, which is then activated via a command line option when `mvn`
is run by the workflow.

| Repository | Workflow | Maven pom.xml | 
| ----- | ----- | -----|
| [Chips-n-Salsa](https://github.com/cicirello/Chips-n-Salsa) | [build.yml](https://github.com/cicirello/Chips-n-Salsa/blob/master/.github/workflows/build.yml) | [pom.xml](https://github.com/cicirello/Chips-n-Salsa/blob/master/pom.xml) |
| [JavaPermutationTools](https://github.com/cicirello/JavaPermutationTools) | [build.yml](https://github.com/cicirello/JavaPermutationTools/blob/master/.github/workflows/build.yml) | [pom.xml](https://github.com/cicirello/JavaPermutationTools/blob/master/pom.xml) |
| [ZigguratGaussian](https://github.com/cicirello/ZigguratGaussian) | [build.yml](https://github.com/cicirello/ZigguratGaussian/blob/master/.github/workflows/build.yml) | [pom.xml](https://github.com/cicirello/ZigguratGaussian/blob/master/pom.xml) |

## License

This GitHub action is released under
the [MIT License](https://github.com/cicirello/jacoco-badge-generator/blob/main/LICENSE).
