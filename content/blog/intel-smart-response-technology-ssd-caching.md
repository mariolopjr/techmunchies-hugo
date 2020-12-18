---
title: Intel Smart Response Technology SSD Caching
date: '2014-01-18T09:44:37-05:00'
description: >-
  Unable to enable Intel Rapid Storage Technology? Read on for a possible
  solution.
---
For those who are having issues with enabling Intel’s SRT SSD caching (where the performance tab does not show up even though you have a compatible board, with both the SSD/HDD plugged into an Intel controller with the latest Intel Rapid Storage Technology software installed), try shrinking the last partition on your disk by ~200MBs. Check the IRST software. It should now allow you to accelerate (assuming that your system meets all other requirements AND the SSD does NOT have any partitions on it, as well as an MBR partition table).

Once you have enabled acceleration, you should see the free space drop from ~200MB to 196-198MB. You can now expand the last partition, it should not affect the SSD caching.

Let me know via email if you are still experiencing issues (there’s a lot of things that need to be right in order for you to enable Intel’s SSD caching).
