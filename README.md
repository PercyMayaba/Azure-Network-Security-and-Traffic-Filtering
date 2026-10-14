I completed this exercise using Cloud Security Engineering lab, I focused on understanding how network security is implemented in Microsoft Azure. My goal was to learn how Azure virtual networks, subnets, and Network Security Groups work together to control network traffic and reduce unnecessary exposure.g the Azure Portal and Azure Cloud Shell with Bash and Azure CLI. I used my existing BonganiSecurity resource group to keep the lab organised and to build on the identity security work I completed on Day 1.

This exercise helped me understand that cloud security is not only about protecting user accounts and controlling permissions. It is also about controlling how network traffic reaches cloud resources and making sure that services are not unnecessarily exposed.

1. Understanding the Azure Network Environment
<img width="2528" height="1696" alt="azure-diagram-1-environment-overview (2)" src="https://github.com/user-attachments/assets/30a227f9-220d-4db1-846d-326069dbbd0b" />


Before creating the network components, I confirmed that my BonganiSecurity resource group was available and that its provisioning state was Succeeded. I also retrieved the Azure region associated with the resource group so that I could create the network resources in the same location.

I used Azure Cloud Shell to interact with my environment through the command line. This gave me practical experience using Azure CLI commands to inspect resources, create network components, and verify their configuration.

Starting with an existing resource group also helped me keep the project structured. In a larger cloud environment, organising resources into logical groups makes it easier to manage configurations, investigate issues, and understand which resources belong to a particular application or workload.

2. Creating the Virtual Network
 <img width="1920" height="1280" alt="azure-diagram-2-virtual-network" src="https://github.com/user-attachments/assets/433d92bd-e2b2-4c28-9c62-7a8d80117c93" />

  

The next step was to create a virtual network named vnet-bongani-day2. In Azure, a virtual network provides a private networking environment where Azure resources can communicate according to the network configuration and security controls applied to them.

I configured the virtual network with the address space 10.20.0.0/16. I selected a private address range to establish an internal network for the lab rather than assigning public IP addresses or exposing a workload to the internet.

I also created a subnet named subnet-workload with the address range 10.20.1.0/24. A subnet divides a virtual network into smaller address ranges, which helps organise resources and apply network security controls at an appropriate level.

The reason I created a separate subnet was to understand how network segmentation supports security. In a production environment, different application components may be placed in separate subnets to help control communication between workloads and reduce unnecessary access.

The virtual network and subnet provided the foundation for the rest of the exercise. No virtual machine was deployed as part of this initial lab, so the work focused on network configuration rather than testing a live application.

3. Creating a Network Security Group
<img width="1920" height="1280" alt="azure-diagram-3-nsg" src="https://github.com/user-attachments/assets/88b47e70-1fa9-433e-b256-a9307822488b" />

After creating the virtual network and subnet, I created a Network Security Group named nsg-bongani-day2.

A Network Security Group, commonly known as an NSG, is an Azure networking feature that filters inbound and outbound traffic using security rules. These rules can evaluate information such as the source address, destination address, protocol, port, direction, and priority.

I wanted to understand how network security rules can be used to restrict unnecessary access to cloud resources. For example, an organisation may need to allow specific application traffic while preventing unrelated or unauthorised connections from reaching its workloads.

Creating the NSG gave me a practical environment in which to inspect Azure's default security rules and learn how custom rules are evaluated.

4. Associating the NSG with the Subnet
   <img width="1920" height="1280" alt="azure-diagram-4-nsg-association" src="https://github.com/user-attachments/assets/4124f31d-723f-4ed8-a177-d4af5e7336d0" />


Creating an NSG is only part of the configuration. I also needed to associate it with the subnet so that the security rules could apply to matching traffic for resources connected to that subnet.

I associated nsg-bongani-day2 with subnet-workload and then inspected the subnet configuration to check the association.

This step helped me understand the difference between creating a security control and applying it to the intended network location. A security group that exists but is not associated with a relevant network interface or subnet will not provide the intended traffic filtering at that location.

In a production environment, checking these associations is an important part of reviewing network security. Incorrect or missing associations can leave workloads with a different level of protection than an administrator intended.

5. Configuring a Custom Inbound Security Rule
   <img width="2528" height="1696" alt="azure-diagram-5-rule-priority (1)" src="https://github.com/user-attachments/assets/aae54818-5837-41eb-b077-bc94a8900e76" />


To gain practical experience with traffic filtering, I created a custom NSG rule named Deny-Inbound-SSH.

The rule was configured to deny inbound TCP traffic destined for port 22, which is commonly used by Secure Shell, or SSH, for remote administration. I assigned the rule a priority of 200.

I chose a deny rule for this exercise because I wanted to practise restricting an unnecessary inbound connection rather than opening a service to the internet. The rule was configured to match all source addresses, so matching inbound SSH traffic would be denied by this rule wherever the NSG applies and no higher-priority rule takes precedence.

This exercise helped me understand how a security engineer can use network rules to reduce the available attack surface. Remote administration services should only be accessible through approved methods and from appropriately restricted sources.

I did not deploy a virtual machine or establish an actual SSH connection during this exercise. Therefore, I verified the rule configuration in Azure, but I did not claim to have performed a live network connectivity test.

6. Understanding NSG Rule Priority

While configuring the custom rule, I also learned why rule priority matters in Azure NSGs.

Azure evaluates custom security rules according to their numeric priority, with lower numbers evaluated before higher numbers. A matching rule with a higher evaluation priority can determine the outcome before a later rule is considered.

For example, a deny rule with priority 200 is evaluated before a rule with priority 300. However, the actual outcome depends on the complete set of applicable rules, their matching conditions, and whether another applicable network control is involved.

Understanding priority is important because a rule may appear correct when viewed individually but behave differently when considered alongside other rules. This is why reviewing the full rule list is more useful than checking only the custom rule.

The exercise gave me a practical introduction to reviewing security rules as a complete configuration rather than treating each rule in isolation.

7. Verifying the Network Security Configuration

After creating the custom rule, I used Azure CLI to inspect its configuration and confirm that the intended settings were present.

I checked the rule name, priority, direction, access setting, protocol, and destination port. I also inspected the subnet configuration to confirm that the NSG was associated with subnet-workload.

This verification process was important because making a configuration change is not the same as confirming that the intended configuration exists. In a cloud security role, I would want to retain evidence that a control was configured correctly and investigate any difference between the intended state and the actual state.

The configuration review was the main validation method used in this lab. Since no virtual machine or other test workload was deployed, I did not perform a packet-level test or attempt to establish an SSH connection.

This distinction is important when documenting security work. A configuration review can demonstrate that a rule exists, while a connectivity test provides additional evidence of how traffic behaves in a working environment.

8. Security Findings

Finding 1: Network segmentation

The virtual network and subnet provided a structured foundation for organising Azure resources. Subnets can help separate workloads and create appropriate locations for network security controls.

My recommendation is to design network segments around application requirements and trust boundaries, rather than placing every workload into a single unrestricted network segment.

Finding 2: Inbound SSH restriction

The custom NSG rule was configured to deny inbound TCP traffic to port 22. This demonstrates how network rules can be used to restrict a common remote administration service.

My recommendation is to avoid exposing SSH broadly to the public internet. Where remote administration is necessary, access should be designed around approved administrative paths and restricted source ranges.

Finding 3: Security control association

The NSG was intended to protect the workload subnet. Verifying the association was therefore an important part of the lab.

My recommendation is to review both security rules and their associations during cloud security assessments. The presence of an NSG alone does not prove that the intended resources are protected by it.

Finding 4: Configuration verification

I used Azure CLI to inspect the security rule and network configuration after making changes.

My recommendation is to maintain clear evidence of security configurations and distinguish configuration checks from live security testing. This makes technical reports more accurate and easier for other engineers to review.

9. Skills Practised

Through this exercise, I gained hands-on practice with Azure virtual networks, subnet address ranges, Network Security Groups, custom traffic-filtering rules, and Azure CLI.

I also strengthened my understanding of network segmentation, inbound traffic control, security rule priority, and the importance of validating a security control after configuring it.

These concepts form part of the foundation for more advanced cloud security work, including network access reviews, secure workload deployment, incident investigation, and cloud security posture assessments.

10. Limitations and Next Steps

This was a foundational configuration lab. I created the network components and configured a custom NSG rule, but I did not deploy a virtual machine or test traffic against a running workload.

My next step would be to extend the exercise with a controlled test workload, subject to budget and security considerations. I could then test permitted and denied connections, inspect the effective network security configuration, and collect evidence showing the difference between configuration validation and observed network behaviour.

I would also explore how NSGs interact with other Azure network controls and how network security decisions can be documented as part of a broader cloud security assessment.

This lab helped me move beyond the theory of Azure networking and practise configuring foundational network security controls. I learned how virtual networks and subnets organise cloud resources, how Network Security Groups filter traffic, and why rule priority and subnet association matter.

The biggest lesson for me was that network security depends on both the rules an engineer creates and where those rules are applied. A well-designed rule is only useful when it is associated with the correct network boundary and its behaviour is verified appropriately.

I will build on these skills in the next stage of my Azure Cloud Security Engineering lab series, where I plan to explore Azure security posture management, threat detection, identity protection, and monitoring.

Disclaimer

This project is a personal Azure learning lab. It demonstrates foundational network security configuration and verification. It is not a formal security audit, penetration test, or production deployment.
