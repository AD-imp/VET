# 文档

假设我们已经在TOLLER's repo的根目录中名为' TRACE_ID '的文件夹中收集了一个跟踪，就可以使用以下命令来处理该跟踪:

* 首先，将具体的UI层次结构转换为抽象的UI状态。

  ```bash
  $ python3 useful-scripts-vet/preprocess-trace.py {TRACE_ID}
  ```

  现在应该看到一个名为' processed_traces '的文件夹，其中有一个名为' {TRACE_ID}.pickle '的新文件。
* 然后，预先计算抽象UI状态之间的相似性。

  ```bash
  $ python3 useful-scripts-vet/preprocess-sim.py {TRACE_ID}
  ```

  现在应该看到一个名为' processed_sims '的文件夹，其中有一个名为' {TRACE_ID}.pickle '的新文件。
* 分区检测

  ```bash
  $ python3 useful-scripts-vet/detect-partition.py {TRACE_ID}
  ```

  现在应该看到一个名为“detect_partition”的文件夹，其中有一个名为“{TRACE_ID}.json”的新文件。
* 过度局部探测

  ```bash
  $ python3 useful-scripts-vet/detect-trapped.py {TRACE_ID}
  ```

  现在应该看到一个名为' detect_trapped '的文件夹，其中有一个名为' {TRACE_ID}.json '的新文件。

由两个检测算法生成的每个JSON文件包含一个检测区域列表，每个区域的格式为“[{TS_ACTION}， {TS_REGION_BEGIN}， {TS_REGION_END}]”。见“{TRACE_ID} / {TS_ACTION}。json '为UI层次结构的动作，算法认为已经导致了相应类型的勘探tarpit。

假设对于某些(工具、应用程序)对，您有一个或多个带有检测区域的跟踪。现在是时候根据它们的长度对这些区域进行排名了(正如文章中提到的):

```bash
$ CT_REGIONS=3 python3 useful-scripts-vet/generate-actions.py "{TOOL_ID}-{APP_ID}-top3" {TRACE_ID1} {TRACE_ID2} ...
```

现在，应该看到一个名为“prevent_actions”的文件夹，其中有一个名为“{TOOL_ID}-{APP_ID}-top3.json”的新文件。该JSON文件的格式与`regions.tar `文件的格式相同。

最后，可以告诉测试记录器(控制TOLLER)在新的运行中强制执行被阻止的操作列表:

```bash
$ XPATHS_TO_KILL=$(python3 useful-scripts-vet/combine-actions.py "{TOOL_ID}-{APP_ID}-top3" | xargs python3 test-recorder/get-source-xpath.py) \
  WATCHDOG_INTV=30000 CTRL_PORT={CTRL_PORT} MINICAP_PORT={MINICAP_PORT} \
    java -jar test-recorder/TestRecorder.jar {OUT_DIR} {SCREEN_OUT_DIR}
```
