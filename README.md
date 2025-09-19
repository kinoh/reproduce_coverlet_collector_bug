# reproduce_coverlet_collector_bug

This repository demonstrates a known bug in Coverlet where XPlat Code Coverage collector fails with file locking errors when a project has a library named "Collector".

## Known Bug Details

- **Issue**: [coverlet-coverage/coverlet#1739](https://github.com/coverlet-coverage/coverlet/issues/1739)
- **Related Issues**: [#857](https://github.com/coverlet-coverage/coverlet/issues/857), [#725](https://github.com/coverlet-coverage/coverlet/issues/725)
- **Status**: Unresolved bug since 2020
- **Cause**: Name conflict between Coverlet internal "Collector.dll" and user project assemblies

## Recommended Solution

**Use Microsoft's standard Code Coverage tool instead of Coverlet:**

```bash
# Coverlet (has bug)
dotnet test --collect:"XPlat Code Coverage"

# Microsoft Code Coverage (recommended)
dotnet test --collect:"Code Coverage;Format=Cobertura"
```

### GitHub Actions Example

```yaml
- name: Test with Coverage
  run: dotnet test --collect:"Code Coverage;Format=Cobertura" --results-directory ./coverage

- name: Code Coverage Summary
  uses: irongut/CodeCoverageSummary@v1.3.0
  with:
    filename: coverage/**/*.cobertura.xml
    badge: true
    format: markdown
```

## How to reproduce

1. `dotnet build Test/Test.csproj`
1. `dotnet test Test/Test.csproj --collect:"XPlat Code Coverage"`

```
$ dotnet build Test/Test.csproj
  Determining projects to restore...
  Restored C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Collector\Collector.csproj (in 83 ms).
  Restored C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\Test.csproj (in 404 ms).
  Collector -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Collector\bin\Debug\net8.0\Collector.dll
  Test -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Test.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:03.29

$ dotnet test Test/Test.csproj --collect:"XPlat Code Coverage"
  Determining projects to restore...
  All projects are up-to-date for restore.
  Collector -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Collector\bin\Debug\net8.0\Collector.dll
  Test -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Test.dll
Test run for C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Test.dll (.NETCoreApp,Version=v8.0)
VSTest version 17.11.1 (x64)

Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
Data collector 'XPlat code coverage' message: [coverlet]Coverlet.Collector.Utilities.CoverletDataCollectorException: CoverletCoverageDataCollector: Failed to instrument modules
 ---> System.AggregateException: One or more errors occurred. (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.) (The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.)
 ---> System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55
   --- End of inner exception stack trace ---
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 62
   at Coverlet.Core.Helpers.RetryHelper.Retry(Action action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 26
   at Coverlet.Core.Helpers.InstrumentationHelper.RestoreOriginalModule(String module, String identifier) in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 267
   at Coverlet.Core.Coverage.PrepareModules() in /_/src/coverlet.core/Coverage.cs:line 144
   at Coverlet.Collector.DataCollection.CoverageWrapper.PrepareModules(Coverage coverage) in /_/src/coverlet.collector/DataCollection/CoverageWrapper.cs:line 71
   at Coverlet.Collector.DataCollection.CoverageManager.InstrumentModules() in /_/src/coverlet.collector/DataCollection/CoverageManager.cs:line 66
 ---> (Inner Exception #1) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #2) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #3) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #4) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #5) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #6) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #7) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #8) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #9) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #10) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

 ---> (Inner Exception #11) System.IO.IOException: The process cannot access the file 'C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Collector.dll' because it is being used by another process.
   at System.IO.FileSystem.CopyFile(String sourceFullPath, String destFullPath, Boolean overwrite)
   at Coverlet.Core.Helpers.FileSystem.Copy(String sourceFileName, String destFileName, Boolean overwrite) in /_/src/coverlet.core/Helpers/FileSystem.cs:line 35
   at Coverlet.Core.Helpers.InstrumentationHelper.<>c__DisplayClass17_0.<RestoreOriginalModule>b__0() in /_/src/coverlet.core/Helpers/InstrumentationHelper.cs:line 269
   at Coverlet.Core.Helpers.RetryHelper.<>c__DisplayClass0_0.<Retry>b__0() in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 28
   at Coverlet.Core.Helpers.RetryHelper.Do[T](Func`1 action, Func`1 backoffStrategy, Int32 maxAttemptCount) in /_/src/coverlet.core/Helpers/RetryHelper.cs:line 55<---

   --- End of inner exception stack trace ---
   at Coverlet.Collector.DataCollection.CoverageManager.InstrumentModules() in /_/src/coverlet.collector/DataCollection/CoverageManager.cs:line 70
   at Coverlet.Collector.DataCollection.CoverletCoverageCollector.OnSessionStart(Object sender, SessionStartEventArgs sessionStartEventArgs) in /_/src/coverlet.collector/DataCollection/CoverletCoverageCollector.cs:line 143.

Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: 49 ms - Test.dll (net8.0)
```

If you rename Collector to something like Collectxr, it works correctly.

```
$ dotnet build Test/Test.csproj
  Determining projects to restore...
  Restored C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Collector\Collectxr.csproj (in 100 ms).
  Restored C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\Test.csproj (in 394 ms).
  Collectxr -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Collector\bin\Debug\net8.0\Collectxr.dll
  Test -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Test.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:03.16

$ dotnet test Test/Test.csproj --collect:"XPlat Code Coverage"
  Determining projects to restore...
  All projects are up-to-date for restore.
  Collectxr -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Collector\bin\Debug\net8.0\Collectxr.dll
  Test -> C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Test.dll
Test run for C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\bin\Debug\net8.0\Test.dll (.NETCoreApp,Version=v8.0)
VSTest version 17.11.1 (x64)

Starting test execution, please wait...
A total of 1 test files matched the specified pattern.

Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: 28 ms - Test.dll (net8.0)

Attachments:
  C:\Users\<redacted>\src\reproduce_coverlet_collector_bug\Test\TestResults\43848166-1579-472d-b197-4800c0eea132\coverage.cobertura.xml
```
