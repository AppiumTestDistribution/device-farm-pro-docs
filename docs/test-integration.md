---
title: Test Integration
---

## Setting Test Status

To reflect the test status on dashboard.

### WDIO

```
      await driver.executeScript('devicefarm: setSessionStatus', [
        {
          status: 'passed', //passed or failed
        },
      ]);
```

## Setting Test Name

To reflect the test name on dashboard.

### WDIO

```
      await driver.executeScript('devicefarm: setSessionName', [
        {
          name: 'Test Name',
        },
      ]);
```
