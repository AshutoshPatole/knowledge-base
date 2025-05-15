# Purpose of Terraform State
State is a necessary requirement for [[terraform]] to function. 

Terraform requires some sort of database to map Terraform config to the real world. When you have a resource `resource "aws_instance" "foo"` in your configuration, Terraform uses this map to know that instance `i-abcd1234` is represented by that resource.

For some providers like AWS, Terraform could theoretically use something like AWS tags. Early prototypes of Terraform actually had no state files and used this method. However, we quickly ran into problems. The first major issue was a simple one: not all resources support tags, and not all cloud providers support tags.

Therefore, for mapping configuration to resources in the real world, Terraform uses its own state structure.