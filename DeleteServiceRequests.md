```cs
Eval 11112 into MyReferenceId

Query  RequestHandling.svc/SrvRequestSet Select "ReqId,RowState,CancelAllScope" Filter with Json Into AllRequestIdsToBeCancelled
{
    "ReferenceId" : "{$MyReferenceId}",
    "RowState" : "!= Cancelled"
    
}

Eval AllRequestIdsToBeCancelled.value.Items(0).ReqId into myReqId

// ###############################################
// Get all Tasks for the request and cancel them
// ###############################################

Query ServiceWorkHandling.svc/JtTaskSet Select "TaskSeq,Objstate" Filter with Json Into AllTasksIdsToBeCancelled 
{
    "ReqId" : "{$myReqId}"
}

Print AllTasksIdsToBeCancelled

Eval AllTasksIdsToBeCancelled.value.ItemsCount() into totTasks
Eval null into firstTask
Eval AllTasksIdsToBeCancelled.value.Items(0).TaskSeq into firstTask  when (totTasks==1 || totTasks==2) && AllTasksIdsToBeCancelled.value.Items(0).Objstate != "CANCELLED"
Eval AllTasksIdsToBeCancelled.value.Items(1).TaskSeq into secondTask when totTasks==2 && AllTasksIdsToBeCancelled.value.Items(1).Objstate != "CANCELLED"

// Cancel the first Task
Get ServiceWorkHandling.svc/JtTaskSet(TaskSeq={$firstTask}) into TaskDetailsOne when (totTasks==1 || totTasks==2) && AllTasksIdsToBeCancelled.value.Items(0).Objstate != "CANCELLED"
Post ServiceWorkHandling.svc/JtTaskSet(TaskSeq={$firstTask})/IfsApp.ServiceWorkHandling.JtTask_Cancel using TaskDetailsOne into Releaseresponse when (totTasks==1 || totTasks==2) && AllTasksIdsToBeCancelled.value.Items(0).Objstate != "CANCELLED"
{
}

// Cancel the second Task
Get ServiceWorkHandling.svc/JtTaskSet(TaskSeq={$secondTask}) into TaskDetailsOne when totTasks==2 && AllTasksIdsToBeCancelled.value.Items(1).Objstate != "CANCELLED"
Post ServiceWorkHandling.svc/JtTaskSet(TaskSeq={$secondTask})/IfsApp.ServiceWorkHandling.JtTask_Cancel using TaskDetailsOne into Releaseresponse when totTasks==2 && AllTasksIdsToBeCancelled.value.Items(1).Objstate != "CANCELLED"
{
}

// ###############################################
// Cancel the Scope of the Request
// ###############################################

Query  RequestHandling.svc/SrvRequestScopeSet Select "SrvRequestScopeId,Objstate" Filter with Json Into ScopeToBeCancelled
{
    "ReqId" : "{$myReqId}"
    
}

Eval ScopeToBeCancelled.value.Items(0).SrvRequestScopeId into scopeId  when  ScopeToBeCancelled.value.Items(0).Objstate != "Cancelled"

// Cancel the first scope
Get  RequestHandling.svc/SrvRequestScopeSet(SrvRequestScopeId={$scopeId}) into Scope when  ScopeToBeCancelled.value.Items(0).Objstate != "Cancelled"
Post RequestHandling.svc/SrvRequestScopeSet(SrvRequestScopeId={$scopeId})/IfsApp.RequestHandling.SrvRequestScope_Cancel using Scope into ScopeResponse when  ScopeToBeCancelled.value.Items(0).Objstate != "Cancelled"
{
}


// ###############################################
// Finally Cancel the Request
// ###############################################
Get RequestHandling.svc/SrvRequestSet(ReqId='{$myReqId}') into RequestDetails

Post RequestHandling.svc/SrvRequestSet(ReqId='{$myReqId}')/IfsApp.RequestHandling.SrvRequest_Cancel using RequestDetails into Releaseresponse
{
}

```