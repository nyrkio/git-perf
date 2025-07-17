# git-perf - Your git performance history

Extends git log, git blame, git status, git show and git diff with performance change data from nyrkio.com

This is a work in progress, currently `git perf blame` `git perf log` and `git perf status` are supported.

## Installation

    curl https://raw.githubusercontent.com/nyrkio/git-perf/refs/heads/main/src/git-perf > $HOME/bin/git-perf
    chmod a+x $HOME/bin/git-perf



## Example

    $ git perf log
    ...
    commit f24e254ec6c77e166791461111c7747f33334f0a
    Author: Pekka Enberg <penberg@iki.fi>
    Date:   Thu Jul 10 14:28:38 2025 +0300

        core/translate: Fix "misuse of aggregate function" error message
        
        ```
        sqlite> CREATE TABLE test1(f1, f2);
        sqlite> SELECT SUM(min(f1)) FROM test1;
        Parse error: misuse of aggregate function min()
        SELECT SUM(min(f1)) FROM test1;
                    ^--- error here
        ```
        
        Spotted by SQLite TCL tests.

    commit 6749af7037f9f1895dedff45dd28ab08f40f9a9f
    Merge: 7a259957 89b0574f
    Author: Pekka Enberg <penberg@iki.fi>
    Date:   Thu Jul 10 14:02:57 2025 +0300

        Merge 'core/translate: Return error if SELECT needs tables and there are non
        
        Fixes https://github.com/tursodatabase/turso/issues/1972
        
        Reviewed-by: Jussi Saurio <jussi.saurio@gmail.com>
        
        Closes #2023

      perf change:           https://nyrk.io/gh/tursodatabase%252Fturso/main
          turso/main/Prepare__SELECT_1_/limbo_parse_query/SELECT_1:
            time: +64.63 %   1.2342->2.0319

    commit 7a259957ac6fc1fe3484a5356968fd3a70c61c96
    Merge: 1333fc88 832f9fb8
    Author: Pekka Enberg <penberg@iki.fi>
    Date:   Thu Jul 10 13:54:15 2025 +0300

        Merge 'properly set last_checksum after recovering wal' from Pere Diaz Bou
        
        We store `last_checksum` to do cumulative checksumming. After reading
        wal for recovery, we didn't set last checksum properly in case there
        were no frames so this cause us to not initialize last_checksum
        properly.
        
        Reviewed-by: Jussi Saurio <jussi.saurio@gmail.com>
        
        Closes #2030

The middle commit introduced a +64% perf regression for one benchmark.


    $ git perf blame core/storage/btree.rs
    d688cfd54↑ core/storage/btree.rs (pedrocarlo       2025-05-31 23:24:27 -0300    1) use tracing::{instrument, Level};
    d688cfd54↑ core/storage/btree.rs (pedrocarlo       2025-05-31 23:24:27 -0300    2) 
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300    3) use crate::{
    b1073da4a  core/storage/btree.rs (Jussi Saurio     2025-04-13 15:17:55 +0300    4)     schema::Index,
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300    5)     storage::{
    133d49872  core/storage/btree.rs (Jussi Saurio     2025-06-18 12:17:48 +0300    6)         header_accessor,
    5827a3351  core/storage/btree.rs (Zaid Humayun     2025-05-28 20:39:18 +0530    7)         pager::{BtreePageAllocMode, Pager},
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300    8)         sqlite3_ondisk::{
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300    9)             read_u32, read_varint, BTreeCell, PageContent, PageType, TableInteriorCell,
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300   10)             TableLeafCell,
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300   11)         },
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300   12)     },
    5bd47d746↓ core/storage/btree.rs (pedrocarlo       2025-05-16 14:38:01 -0300   13)     translate::{collate::CollationSeq, plan::IterationDirection},
    fa442ecd6  core/storage/btree.rs (Pekka Enberg     2025-07-03 13:13:58 +0300   14)     turso_assert,
    b0c64cb4d↑ core/storage/btree.rs (Pere Diaz Bou    2025-06-03 14:28:19 +0200   15)     types::{IndexKeyInfo, IndexKeySortOrder, ParseRecordState},
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300   16)     MvCursor,
    51ad827f1  core/storage/btree.rs (Pere Diaz Bou    2024-11-19 17:56:24 +0100   17) };
    7cb7eb4e6  core/storage/btree.rs (Krishna Vishal   2025-02-07 01:15:21 +0530   18) 
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300   19) use crate::{
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300   20)     return_corrupt,
    e3f71259d↓ core/storage/btree.rs (Pekka Enberg     2025-05-15 09:41:59 +0300   21)     types::{compare_immutable, CursorResult, ImmutableRecord, RefValue, SeekKey, SeekOp, Value},
    11782cbff  core/storage/btree.rs (Pekka Enberg     2025-04-10 07:52:10 +0300   22)     LimboError, Result,
    8642d416c⇊ core/storage/btree.rs (Pere Diaz Bou    2025-03-25 09:49:22 +0100   23) };
    05621e328  core/btree.rs         (Pekka Enberg     2023-08-30 20:24:22 +0300   24) 
    99e0cf060⇈ core/storage/btree.rs (meteorgan        2025-07-08 22:55:25 +0800   25) use super::{
    99e0cf060⇈ core/storage/btree.rs (meteorgan        2025-07-08 22:55:25 +0800   26)     pager::PageRef,
    99e0cf060⇈ core/storage/btree.rs (meteorgan        2025-07-08 22:55:25 +0800   27)     sqlite3_ondisk::{
    99e0cf060⇈ core/storage/btree.rs (meteorgan        2025-07-08 22:55:25 +0800   28)         write_varint_to_vec, IndexInteriorCell, IndexLeafCell, OverflowCell, DATABASE_HEADER_SIZE,
    99e0cf060⇈ core/storage/btree.rs (meteorgan        2025-07-08 22:55:25 +0800   29)         MINIMUM_CELL_SIZE,
    99e0cf060⇈ core/storage/btree.rs (meteorgan        2025-07-08 22:55:25 +0800   30)     },
    99e0cf060⇈ core/storage/btree.rs (meteorgan        2025-07-08 22:55:25 +0800   31) };

The commits that caused performance changes are decorated with a up ↑ or down ↓ arrow. A double arrow means the change was larger than 10%. Note however that often there are multiple tests that fail and in that case it is arbitrary which one was used to draw the arrow symbol.

    $ git perf status
    On branch main
    Your branch is up to date with 'upstream/main'.

    nothing to commit, working tree clean

    git@github.com:tursodatabase/turso.git/main: 208 commits flagged as either a regression or improvement
                    The most recent perf change is cb163fc at 2025-07-17 13:56:49: sys:0.001
                    290 metrics across 148 tests reported 1686 changes.
                    The test 'clickbench/limbo/main/29_SELECT_Title__COUNT____AS_PageViews_FROM_hits_W' caused 19 of above changes.
                    For more details, see https://nyrk.io/gh/tursodatabase%252Fturso/main

# Benchmarking Howto

To get to the point where the above works, you need to add some benchmarking to your CI workflow.
Checkout [nyrkio/change-detection](https://github.com/marketplace/actions/nyrkio-change-detection)
github action for an easy way to capture bencchmark results and then send them to Nyrkiö for change
point detection.


# Open Source

Copyright 2025 Nyrkiö Oy

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
