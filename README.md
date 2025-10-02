# mr-cancelation-8.8

A minimal reproducer for a Camunda 8.8.0 process cancelation issue.

## To Run

Start the camunda/camunda:8.8-SNAPSHOT image locally:

```bash
docker compose -d
```

Wait a minute for it to start. 

Run the reproducer: 

```bash
./test.sh
```

## Observed output

```bash
 ./test.sh
Deployment Response:
{
  "tenantId": "<default>",
  "deploymentKey": "2251799813685668",
  "deployments": [
    {
      "processDefinition": {
        "processDefinitionId": "test-process",
        "processDefinitionVersion": 2,
        "resourceName": "model.bpmn",
        "tenantId": "<default>",
        "processDefinitionKey": "2251799813685630"
      }
    }
  ]
}
Process Definition Key: 2251799813685630
Response (by key as string)
{
  "processDefinitionId": "test-process",
  "processDefinitionVersion": 2,
  "tenantId": "<default>",
  "variables": {},
  "processDefinitionKey": "2251799813685630",
  "processInstanceKey": "2251799813685669",
  "tags": []
}
Process Instance Key: 2251799813685669
Calling cancellation for processInstanceKey=2251799813685370
Cancellation failed, HTTP 500
{
  "type": "about:blank",
  "title": "INTERNAL",
  "status": 500,
  "detail": "Command 'CANCEL' rejected with code 'PROCESSING_ERROR': Expected to process command 4503599627370618 (PROCESS_INSTANCE.CANCEL) without errors, but unexpected error occurred. Check your broker logs (partition 1), or ask your operator, for more details.",
  "instance": "/v2/process-instances/2251799813685370/cancellation"
}
```

## Broker log

```
{"timestampSeconds":1759374934,"timestampNanos":8214590,"severity":"ERROR","message":"Expected to process record 'TypedRecordImpl{metadata=RecordMetadata{recordType=COMMAND, valueType=PROCESS_INSTANCE, intent=CANCEL, authorization={\"format\":\"UNKNOWN\",\"authData\":\"***\",\"claims\":\"***\"}}, value={\"bpmnElementType\":\"UNSPECIFIED\",\"elementId\":\"\",\"bpmnProcessId\":\"\",\"version\":-1,\"processDefinitionKey\":-1,\"processInstanceKey\":-1,\"flowScopeKey\":-1,\"bpmnEventType\":\"UNSPECIFIED\",\"parentProcessInstanceKey\":-1,\"parentElementInstanceKey\":-1,\"tenantId\":\"<default>\",\"elementInstancePath\":[],\"processDefinitionPath\":[],\"callingElementPath\":[],\"tags\":[]}}' without errors, but exception occurred with message 'Cannot invoke \"io.camunda.zeebe.engine.state.instance.ElementInstance.isTerminating()\" because \"processInstance\" is null'.","logging.googleapis.com/sourceLocation":{"file":"Engine.java","line":246,"function":"io.camunda.zeebe.engine.Engine.handleUnexpectedError"},"logging.googleapis.com/labels":{"actor-name":"StreamProcessor-1","actor-scheduler":"Broker-0","partitionId":"1"},"threadContext":{"id":36,"name":"zb-actors-0","priority":5},"loggerName":"io.camunda.zeebe.broker.process","serviceContext":{"service":"","version":""},"@type":"type.googleapis.com/google.devtools.clouderrorreporting.v1beta1.ReportedErrorEvent","exception":"java.lang.NullPointerException: Cannot invoke \"io.camunda.zeebe.engine.state.instance.ElementInstance.isTerminating()\" because \"processInstance\" is null\n\tat io.camunda.zeebe.engine.processing.bpmn.behavior.BpmnStateTransitionBehavior.executeRuntimeInstructions(BpmnStateTransitionBehavior.java:499)\n\tat io.camunda.zeebe.engine.processing.bpmn.task.JobWorkerTaskProcessor.finalizeTermination(JobWorkerTaskProcessor.java:134)\n\tat io.camunda.zeebe.engine.processing.bpmn.task.JobWorkerTaskProcessor.finalizeTermination(JobWorkerTaskProcessor.java:28)\n\tat io.camunda.zeebe.engine.processing.bpmn.BpmnStreamProcessor.processEvent(BpmnStreamProcessor.java:187)\n\tat io.camunda.zeebe.engine.processing.bpmn.BpmnStreamProcessor.lambda$processRecord$0(BpmnStreamProcessor.java:121)\n\tat io.camunda.zeebe.util.Either$Right.ifRightOrLeft(Either.java:438)\n\tat io.camunda.zeebe.engine.processing.bpmn.BpmnStreamProcessor.processRecord(BpmnStreamProcessor.java:118)\n\tat io.camunda.zeebe.engine.Engine.process(Engine.java:179)\n\tat io.camunda.zeebe.stream.impl.ProcessingStateMachine.batchProcessing(ProcessingStateMachine.java:373)\n\tat io.camunda.zeebe.stream.impl.ProcessingStateMachine.lambda$processCommand$3(ProcessingStateMachine.java:281)\n\tat io.camunda.zeebe.db.impl.rocksdb.transaction.ZeebeTransaction.run(ZeebeTransaction.java:128)\n\tat io.camunda.zeebe.stream.impl.ProcessingStateMachine.processCommand(ProcessingStateMachine.java:281)\n\tat io.camunda.zeebe.stream.impl.ProcessingStateMachine.tryToReadNextRecord(ProcessingStateMachine.java:239)\n\tat io.camunda.zeebe.scheduler.ActorJob.invoke(ActorJob.java:84)\n\tat io.camunda.zeebe.scheduler.ActorJob.execute(ActorJob.java:43)\n\tat io.camunda.zeebe.scheduler.ActorTask.execute(ActorTask.java:125)\n\tat io.camunda.zeebe.scheduler.ActorThread.executeCurrentTask(ActorThread.java:127)\n\tat io.camunda.zeebe.scheduler.ActorThread.doWork(ActorThread.java:104)\n\tat io.camunda.zeebe.scheduler.ActorThread.run(ActorThread.java:224)\n"}
```