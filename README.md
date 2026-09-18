# claude-health-monitoring

Tools for monitoring Claude's activity, performance, and reliability.

## Performance monitoring

[PerfMon: BlackBox performance collection](Perfmon/README.md) documents setup,
the system and Claude counter groups, log inspection, and how to add counters.

The native Windows Performance Monitor collector requests 16 system counter paths
and five Claude-related paths, sampling available counters every 15 seconds.
