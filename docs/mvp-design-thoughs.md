# Arbiter - MVP Design Thoughts 
                                                                                    December 23, 2020  
  

## Overview

Arbiter will be a new, general purpose exemptions management service for centrally managing governance and compliance exemptions within an AWS organization.

AWS services such as AWS Config and AWS Systems Manager will be able to call out to Arbiter to perform exemption lookups to determine if a given AWS resource has a valid exemption from a compliance rule or governance enforcement action.

## Functional Requirements

1. The service components should support being hosted in a centralized governance account and support IAM authenticated access from subordinate managed AWS accounts.

1. The data model should manage exemptions in a decomposed manner. Exemption criteria and rules will be defined in exemption policies and those policies will be applied to target AWS accounts through exemption policy attachments or associations. 

1. Designated account owners within subordinate managed AWS accounts should be able to view details for all governance exemption policies attached to their account. All other authorized users within the account will only be permitted to request the exemption status (e.g. EXEMPT or NON_EXEMPT) for a given AWS resource.

1. Exemption policies should be flexible, with initial support of the following exemption criteria:
    1. Resource tag keys/values
    1. Resource attributes
    1. AWS account ID
    1. AWS account membership in a specific AWS Organizations OU 
    1. Specific attributes assigned to the containing AWS account within AWS Organizations
    1. Resource naming convention
    1. Resource ID
    1. Compliance Rule ID (e.g. AWS Config Rule name)
    1. Governance Action ID (e.g. AWS Systems Manager Document name)
    
1. Exemption policies should support the ability to define time bounded exemption criteria that automatically expire after a specified time period. 

1. Exemption policies and exemption policy attachments should support the ability to expire after a specified time period. Additionally, exemption policy attachments should support the ability to schedule activation and deactivation at a future date/time.

1. Exemption policies should require a mandatory periodic review, support the ability to track the last review date and reviewing authority, and optionally allow exemption policies that exceed the review threshold to be automatically disabled.

1. The service should include the capability to send notifications to exemption policy owners and governance administrators for critical events, monitoring alerts, and policy review reminders. 

1. The service should automatically monitor exemption policy usage and provide the ability to define key metrics, including high and low watermark thresholds for usage over a given time period. It would be nice if the service calculated the average usage for an exemption policy and triggered alerts for anomalous usage patterns.

1. The service should implement key performance metrics and logging, to include a count of the times an exemption policy was applied to a resource, the last date and time an exemption policy was last applied to a resource, and the last AWS account and resource ID to which it was applied.

1. The service should implement a simple REST API for all core functionality.
    1. It may be ideal to separate control plane and data plane APIs and components.

1. Exemption policy attachments should support the ability to be explicitly placed in different states: `enabled`, `disabled`, `evaluation`. If an attachment is disabled, the exemption policy is not evaluated. If an attachment is in evaluation mode, it will be evaluated and the result logged, but the service will ignore the exemption policy when making an actual exemption determination.

1. The service should support integration with AWS Systems Manager Change Calendar to allow governance administrators to define time periods where compliance rules or governance enforcement actions are either prohibited or authorized. 
 
1. **[Optional]** The service should include an emergency "scram" button that will ensure all exemption lookup requests return an emergency exemption to all resource lookups. 

## Potential Issues

1. If we support exemption criteria based on resource tags, any users in subordinate accounts with the adequate permissions to apply or modify resource tags, may be able to bypass critical compliance rules and governance enforcement actions. Can we minimize this exposure by using composite criteria?

  For example: Environment=ProductionCritical and aws:autoscaling:groupName=production-servicea would limit the exemption to only resources with the Environment=ProductionCritical tag *and* that are members of the production-servicea auto scaling group. Since the second tag is an AWS reserved key, users would not be permitted to apply this tag.

## API Actions

- CreateExemptionPolicy **[Restricted Method]**
- CreateExemptionPolicyAttachment **[Restricted Method]**
- DeleteExemptionPolicy **[Restricted Method]**
- DeleteExemptionPolicyAttachment **[Restricted Method]**
- DescribeExemptionPolicy **[Privileged Method]**
- EnableEmergencyBypass **[Restricted Method]**
- GetExemptionStatusForResource
- ListExemptionPolicies **[Restricted Method]**
- ListExemptionPolicyAttachments **[Restricted Method]**
- ListExemptionPoliciesForResource **[Privileged Method]**
- UpdateExemptionPolicy **[Restricted Method]**
- UpdateExemptionPolicyAttachment **[Restricted Method]**
