## Briefing Document: Benchmarking .NET 6 Applications Using BenchmarkDotNet

**Source:** Excerpts from "Benchmarking .NET 6 Applications Using BenchmarkDotNet: A Deep Dive" by Joydip Kanjilal, published in CODE Magazine: 2022 - September/October.

**Date:** January 9, 2025 (Last updated date of source)

**Prepared By:** [Your Name/Organization]

**Purpose:** This briefing document provides a summary and analysis of the key concepts, techniques, and tools for benchmarking .NET 6 applications as presented in the source article.

**Executive Summary:**

The article "Benchmarking .NET 6 Applications Using BenchmarkDotNet: A Deep Dive" by Joydip Kanjilal provides a comprehensive overview of code benchmarking in the context of .NET applications, specifically focusing on .NET 6. It emphasizes the importance of quantifying code performance to identify bottlenecks and areas for optimization. The core of the article is the introduction and practical demonstration of BenchmarkDotNet, an open-source library that facilitates the process of creating, executing, and analyzing performance benchmarks for .NET methods. The article covers the fundamentals of benchmarking, including baselining, and provides detailed examples of benchmarking different aspects of .NET code, such as LINQ performance, StringBuilder usage, and ASP.NET 6 API response times. A real-world use case is presented to illustrate how benchmarking can reveal performance differences between optimized and non-optimized code.

**Key Themes and Important Ideas/Facts:**

- **What is Benchmarking?** Benchmarking is a test that provides "a set of quantifiable results that can help you determine whether an update to your code has increased, decreased, or had no effect on performance." It is essential for understanding the performance metrics of application methods and identifying areas for optimization. Benchmarks can range from broad scope to micro-benchmarks focusing on minor code changes.
- **Why Benchmark Code?** Benchmarking is crucial for "knowing the performance metrics of your application's methods" and for "zeroing in on the parts of the application's code that need reworking." It helps to "identify bottlenecks in an application's performance," which in turn allows developers to "determine the changes required in your source code to improve the performance and scalability of the application."
- **Introducing BenchmarkDotNet:** BenchmarkDotNet is highlighted as a powerful open-source library for .NET that can "convert your .NET methods into benchmarks, monitor those methods, and get insights into the performance data collected." It simplifies the process of transforming methods into benchmarks, running them, and obtaining results. In BenchmarkDotNet terminology, an "operation" is the execution of a method decorated with the [Benchmark] attribute, and a "collection of such operations is known as an iteration."
- **Baselining:** Baselining allows for scaling benchmark results relative to a designated baseline method. By decorating a benchmark method with [Baseline = true], the generated summary report includes a "Ratio" column, with the baseline method having a value of 1.00 and other methods showing their performance relative to this baseline.
- **Benchmarking .NET Code with BenchmarkDotNet:Setup:** Benchmarking with BenchmarkDotNet requires a Console application project in Visual Studio 2022, .NET 6.0, ASP.NET 6.0 Runtime (for ASP.NET benchmarks), and the BenchmarkDotNet NuGet package.
- **Creating Benchmarks:** Benchmarks are created by defining a class with one or more methods decorated with the [Benchmark] attribute. Optional attributes like [GlobalSetup] and [GlobalCleanup] can be used for initialization and cleanup logic that should run only once. [IterationSetup] and [IterationCleanup] can be used for setup and cleanup per iteration.
- **Diagnosers:** Diagnosers provide additional information about benchmarks. The [MemoryDiagnoser] attribute, when attached, provides insights into "allocated bytes and the frequency of garbage collection."
- **Execution:** Benchmarks are executed in Release mode using the BenchmarkRunner.Run method, which can be configured to run on a specific type or an entire assembly. **It is important to note that BenchmarkDotNet currently only supports Console applications for execution.**
- **Interpreting Results:** The benchmark results provide valuable metrics in a table format, including:
- **Method:** The name of the benchmarked method.
- **Mean:** The average execution time.
- **StdDev:** The standard deviation of execution times.
- **Gen 0, Gen 1, Gen 2:** Garbage collection statistics for different generations per 1000 operations.
- **Allocated:** Managed memory allocated per single operation.
- **Rank:** Relative execution speed from fastest to slowest.
- **Benchmarking Examples:** The article provides practical examples demonstrating benchmarking:
- **LINQ Performance:** Comparing the performance of SingleOrDefault and FirstOrDefault methods, showing FirstOrDefault is faster.
- **StringBuilder Performance:** Benchmarking StringBuilder usage with and without the StringBuilderCache, illustrating the performance benefits of using the cache.
- **ASP.NET 6 Application Performance:** Benchmarking the response time of minimal API endpoints using HttpClient and WebApplicationFactory.
- **Real-World Use Case:** A compelling example demonstrates how benchmarking can expose performance differences between an entity with a Guid ID (resource-intensive to generate) and an optimized version using an int ID. Benchmarking the retrieval of lists of these entities clearly shows the performance advantage of the optimized version, highlighting the tool's utility in identifying slow paths and guiding optimization efforts. The GetProductsOptimized method is shown to "consumes less memory and is much faster than its counterpart."
- **Conclusion:** BenchmarkDotNet is presented as a "compelling and easy-to-use framework to benchmark .NET code." It allows for performance testing on various code levels without altering functionality. The article concludes by emphasizing that while benchmarking is a powerful tool, achieving improved performance and scalability ultimately depends on adhering to "best practices."

**Supporting Quotes:**

- "The benchmarking technique helps determine the performance measurements of one or more pieces of code in your application."
- "A benchmark is a simple test that provides a set of quantifiable results that can help you determine whether an update to your code has increased, decreased, or had no effect on performance."
- "Benchmarking code is critical for knowing the performance metrics of your application's methods."
- "Benchmarking also allows you to zero in on the parts of the application's code that need reworking."
- "BenchmarkDotNet is an open-source library compatible with both .NET Framework and .NET Core applications that can convert your .NET methods into benchmarks, monitor those methods, and get insights into the performance data collected."
- "When you decorate a benchmark method with the Baseline attribute and set it to “true,” the summary report generated after the benchmark shows an additional column named “Ratio.”"
- "Note that BenchmarkDotNet works only with Console applications. It won't support ASP.NET 6 or any other application types."
- "As is evident from the benchmark results, the First method is much faster than the Single method in LINQ."
- "As you can see, the GetProductsOptimized consumes less memory and is much faster than its counterpart, i.e., the GetProducts method."
- "BenchmarkDotNet is a compelling and easy-to-use framework to benchmark .NET code."
- "Remember that to improve the performance and scalability of your application, you must adhere to the best practices, if not, merely benchmarking your application's code won't help."

**Actionable Insights/Recommendations:**

- Development teams should integrate benchmarking into their workflows to quantify the performance impact of code changes and identify areas requiring optimization.
- BenchmarkDotNet is a recommended tool for .NET developers seeking to perform detailed performance analysis of their code.
- When using BenchmarkDotNet, it is crucial to run benchmarks in Release mode and utilize the available diagnosers (like MemoryDiagnoser) for comprehensive data.
- Baselining can be a useful technique for comparing the performance of different implementations against a known baseline.
- While BenchmarkDotNet is powerful, it is important to remember that performance optimization also relies on adhering to general .NET best practices.

This briefing document summarizes the key takeaways from the provided source, highlighting the importance and practical application of benchmarking in .NET 6 development using BenchmarkDotNet.