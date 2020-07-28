# Status codes

Every response from the Paysafe Pay Later API contains an object containing the result of the request. In this object the **statusCode** indicates the status of the request. Below an overview of every status code and their description.

Status code | Name | Description
---------|----------|---------
0.0.0 | OK | Operation performed successfully
0.0.1 | OK_DUPLICATE | Duplicate request: Operation already performed successfully
0.0.2 | OK_FINAL_RESULT_PENDING | Operation performed successfully: Final result pending
1.0.0 | PENDING | Pending operation
2.0.0 | DECLINED_RISK_PERMANENTLY | Operation permanently declined
2.1.0 | DECLINED_RISK_RETRYABLE | Operation declined (retryable)
2.1.1 | DECLINED_RISK_RETRYABLE_CUSTOMER_LIMIT | Operation declined (retryable) - limit check failed
3.0.0 | MISSING_INPUT_DATA | Missing mandatory field
3.1.0 | INVALID_INPUT_DATA | Input data is invalid
4.0.0 | WORKFLOW_ERROR | Incorrect workflow state
4.1.0 | WORKFLOW_ERROR_WRONG_PURCHASE_STATE | Transaction State validation failure
4.2.0 | WORKFLOW_ERROR_UNKOWN_REFERENCE | Unknown reference
4.3.0 | WORKFLOW_ERROR_INVALID_PRODUCT | Payment type validation failure
4.4.0 | WORKFLOW_ERROR_DUPLICATE_REQUEST | Duplicate request
4.5.0 | WORKFLOW_ERROR_USER_NOT_AUTHORIZED | User not authorized
4.5.1 | WORKFLOW_ERROR_USER_NOT_AUTHORIZED_PRODUCT_INACTIVE | Channel is not active
5.0.0 | INTERNAL_ERROR | Unexpected internal error
5.1.0 | PROCESSING_SERVICE_UNAVAILABLE | Service currently unavailable
6.0.0 | UNDEFINED_RESULT | Internal error: Operation result undefined