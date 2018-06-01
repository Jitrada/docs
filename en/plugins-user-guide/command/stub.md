% Stub Plugin

Provides a Node Executor, File Copier, and Resource Model Source.  This plugin can be used for testing.

The `stub-plugin` includes these providers:

* `stub` for the NodeExecutor service
* `stub` for the FileCopier service
* `stub` for the ResourceModelSource service

(Refer to [Configuring - Node Execution](configuring.html#node-execution) to enable them.)

This plugin does not actually perform any remote file copy or command execution,
instead it simply echoes the command that was supposed to be executed, and
pretends to have copied a file. 

This is intended for use in testing new Nodes, Jobs or Workflow sequences without
affecting any actual runtime environment.  

You can also test some failure scenarios by configuring the following node attributes:

`stub-exec-success`="true/false"

:   If set to false, the stub command execution will simulate command failure

`stub-result-code`

:   Simulate the return result code from execution

You could, for example, disable or test an entire project's workflows or jobs by
simply setting the `project.properties` node executor provider to `stub`.


File: `rundeck-stub-plugin-${VERSION}.jar`
