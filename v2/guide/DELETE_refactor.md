# CDK refactor and cross-stack references

## Executive summary

CloudFormation’s stack refactoring API does not allow modifications (other than resource moves). But we want to give CDK users a more flexible experience, by allowing them to move resources while also adding and removing other resources. These additions and removals will be ignored by the toolkit, and only what has moved will be sent to the API for refactoring. One implication of this flexibility is that the refactor opeartion may leave the infrastructure in a state that is not the same as the CDK application.

This becomes a problem when this putative intermediate state has cross-stack references that don’t exist in the CDK application. Due to the way CloudFormation deals with references and exports, we would either not be able to create this kind of reference, or create it in a way that blocks subsequent deployments.

The proposed solution in this document is to change the matching algorithm to only allow a refactor if the deployed stack and the CDK output are exactly the same, up to resource locations. So, instead of building a more flexible experience on top of CFN, we impose the same constraints they do. In exchange, we gain predictability and also, when we refuse to do a refactor, it's easy to explain to the user why.

## What's wrong

Suppose you have two resources, with logical IDs `Foo` and `Bar`, living in the same stack. `Foo` has a reference to `Bar`:

```
Resources:
  Foo:
    ...
    Properties:
      SomeProp:
        Ref: Bar # <-- in-stack reference

  Bar: ...
```

Using CloudFormation's stack refactoring API, you can move `Bar` to another stack, as long as you keep `Foo` pointing to it. The way to represent this cross-stack reference in CloudFormation is to create an output in the target stack referencing the resource you want. This output must have an export name which can then be imported using the `Fn::ImportValue` intrinsic function in the consuming stack:

```
# Original stack
Resources:
  Foo:
    ...
    Properties:
      SomeProp:
        Fn::ImportValue: RefBarExport # <-- replaced with a cross-stack one
```

```
# New stack
Resources:
  Bar: ...
Outputs:
  Value:
    Ref: Bar
  Export:
    Name: RefBarExport
```

Converting an in-stack reference into a cross-stack reference like this poses no problems when using CloudFormation templates in isolation. But when the templates are the output of a CDK application, there is a case where we may run into problems. Here is an example:
[Image: Untitled-2025-06-24-1119.excalidraw.png]
Suppose you have a Lambda function and a DynamoDB table in the same stack, with the function referencing the table (via a `Ref`, for example). This is the deployed state of the infrastructure. Then, in their CDK application, the user moves the table to another stack and, at the same time, replaces the function with another one, `Lambda'`. The matching algorithm will detect that the table was moved from Stack X to Stack Y. But since `Lambda'` is different from `Lambda`, the function is not going to be part of the mappings sent to the stack refactoring API. The result of this refactor is that the table will be moved to the other stack, but the function will stay where it was. Since the stack refactoring API doesn’t allow modifications, `Lambda` still needs to reference `Table`, even if they are in different stacks. The natural thing to do would be to convert the `Ref` into an `Fn::ImportValue`, as described above.

But here is the problem: `Fn::ImportValue` needs a corresponding export. And, in this case, there is none. Note how Stack Y, in the CDK output has no need to export anything, even after the move. We could create an ad-hoc export just for the refactor, but that would cause another problem: the next deploy would fail, as it would try to remove the export while an import for it (in `Lambda`) still exists. We could then try to solve that, by doing a corrective deployment immediately after. First deploying Stack X, thus removing the importer, then deploying Stack Y, which would safely remove the export and add the new function. But in the period between the two deploys, there would be no Lambda function at all, potentially causing downtime for the customer.

In summary: **if a refactor would create cross-stack references that don’t exist in the local CDK output, that refactor is invalid, and cannot be executed.**

## Possible solutions

### 1. Break references before deployment

This is a muti-step solution. Before using the stack refactor API, we will first do a deploy to break all references between resources. Here, “breaking a reference” means replacing CloudFormation intrinsic function calls with the value those functions resolve to:

* For `Fn::ImportValue`, we can get the export value from `describeStacks`.
* For `Ref`, we can get the physical ID from `describeStackResources`.
* For `Fn::GetAtt`, we need an additional step. After getting the physical ID from `describeStackResources`, we use CCAPI’s `getResource` method to read the value of the attribute.
* For `Fn::Sub`, we replace each variable that is not in the map, using either the method for `Ref` or `Fn::GetAtt` described in the two previous points.

The next step is the refactor itself. Since there are no references before the refactor, there won’t be any after, either. And then the next deploy is safe because there is no risk of it trying to remove an export that is in use. Note that the next deploy will reinstate all the references that exist in the CDK app. To recap, these are the states the stacks would go through in the process, in the example above:


[Image: states-with-ref-breaking.excalidraw.png]

The problem with this option is that, in the last deployment, if we pick the wrong order, we might cause downtime for customers. In this example, if we deploy stack X before stack Y, there will be a period of time in which no Lambda function is available. This is much higher risk than causing a deployment failure.

### 2. Stop the refactor and give an error message

If we don't have an output to use (and knowing that we can't create one), stop the process, and let the user know that this is an impossible situation. We can marginally reduce the chance of this happening by adding some heuristics to the matching algorithm. These heuristics can exploit patterns in the construct tree. For example, in certain cases we can detect groups of resources that should be moved together because they belong to the same high-level construct.

But there will always be cases that fall outside any of these patterns, such as in the example above.

### 3. Overdo the refactor

This is essentially taking the idea of moving resources together to the extreme, with the following algorithm:

```
1. Produce an initial list of mappings;
2. Flag all the problematic references that would be created by this refactor;
3. While there are flagged references:
4.   For each flagged reference, include its source in the list of mappings;
5.   Re-flag all the references as necessary;
```

Line 4 would guarantee that, instead of generating a problematic cross-stack reference, we would preserve the original in-stack reference (only moved wholesale to another stack). By doing this, however, we could create new problematic references, so we repeat the process until the graph is cleared up.

The downside of this solution is that we may end up refactoring more than the user intended to. For example, moving large numbers of resources across stacks, when the user only wanted a single resource moved.

### 4. Forbid any changes (preferred)

We could go to the other extreme, and forbid any additions, deletions or modifications in the local state compared to the deployed state. We would sacrifice flexibility, but the problem would go away.

### 5. Trust mode for refactoring

The reason this problem arises in the first place is that CloudFormation’s refactor API does not allow modifications. This forces us to compute a new state that includes the resource moves the user has made, but no other changes (additions, deletions or modifications of resources). A successful refactor operation would move the user’s stacks to this new state. But this also creates a mismatch between what is deployed and the CDK output, as in the case of the example above.

If CloudFormation had a “trust mode”, with which clients could include modifications (and therefore CloudFormation made fewer validations), we could send the current state of the CDK application, as is, without having to compute this intermediate state. This trust mode could be enabled with a flag in the API.

*This is the cleanest solution, and the one we would prefer. But most of the work would have to be done on the CloudFormation service. To our knowledge, this is not even in their plans.*

### 6. Direct cross-stack references

Contrary to the old saying, we could solve this problem by *removing* a layer of indirection. Instead of having an output mediating the reference between two resources, CloudFormation could allow direct references, maybe using the existing `Ref` and `Fn::GetAtt` intrinsic functions with fully qualified names (e.g., `{Ref: "SomeStack.SomeResource"}`) or, if necessary, by introducing new functions that work across stacks.

Not only would this simplify the implementation a lot and reduce the risk of bugs, it would also completely solve the problem discussed in this document. If there is no output to create and/or delete, there is no risk of users getting their resources stuck in a deadly embrace. And we could do exactly the refactor the user wants: no more and no less.

The CloudFormation team has already done some [investigation](https://quip-amazon.com/r0TpAvT4CD83/CloudFormation-Cross-AccountRegion-Reference-Solutions) on the problem of cross-stack referencing, and are planning to deliver a new intrinsic function this year. When they deliver this, we can reassess whether we want to switch to the new intrinsic function.
