# Infra (alpha)

Manage AWS infrastructure with Clojure and EDN.

Infra's configuration API preserves AWS's cloudformation options while providing shorthands and utilities that make it easier to configure complex templates.

Note: Currently you can only create resources via the repl but that still beats having to create resources manually via the aws console or having to define configurations via YAML/JSON files. In the future Infra might provide a cli and terminal interface for creating cloudformation stacks.

## Docs

- [Configuring Resources](#configuring-resources)
  - [Resource Types](#resource-types)
  - [Writing Templates](#writing-templates)
- [Creating resources](#creating-resources)

This documentation assumes that you're familiar with [AWS Cloudformation](https://docs.aws.amazon.com/cloudformation/index.html), though you'll mostly just need to reference the [resource property options](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) for the resources you'll be creating.

### Configuring Resources

Your edn file must be a mapping of resource names to their template bodies.

#### Resource Types

A template body can either be a vector, if a template is derived form a url, or a map.

**Template Urls**

Since [AWS Cloudformation Stack options](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_CreateStack.html) are quite verbose, we have a shorthand for defining a template urls. You can declare a tuple where the first arg is the template url and the second is AWS Stack options:

```clj
{:MyStack ["http:/s3.amazonaws.com/path-to-template"
           {:Parameters []}]}
;; {:StackName   "MyStack"
;;  :TemplateURL "http:/s3.amazonaws.com/path-to-template"
;;  :Parameters  []}
```

**Custom Templates**

To declare custom template maps you can pass the [AWS template options](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html) directly. 

For declaring [Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html), we provide a shorthand you can use. Instead of a map with `type` and `properties` key-values, can pass a tuple where the first arg is the resource identifier type keyword (separating the service and its respective module with a period) and the second is its properties.

```clj
{:MyStack 
 {:Resources {:ExResource [:Service.Module
                           {:Name "Foo"}]}}}
;; {:MyStack 
;; {:Resources {:ExResource [{:Type "AWS::Service::Module"
;;                            :Properties {:Name "Foo"}}]}}}
```

#### Writing Templates

Infra provides a few template literals to make it easier to markup your templates.

AWS has [intrinsic function utilities](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html) it uses for marking up JSON. If a reader literal serves a similar purpose as an existing utility we name it after its JSON counterpart.

You can look at the [example configs](https://github.com/rejure/infra.aws/tree/master/example/resources/demo)s to see the template literals listed below in action. 

List of available reader literals: 

* `eid`: append environment info to an identifier string. Useful for naming resources per environment.

* `kvp`: convert key-value parameter map to aws parameter array.

* `ref`: reference a resource by its [logical id](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html).

* `sub`: substitute a keyword with a parameter passed to configuration reader.

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html

### Creating resources

Right now you can only create resources via the repl - but that still beats having to create them via aws console.

Note: If you're creating resources that depend on one another make sure you wait for the given dependencies to finish creating.

Example: 

```clj
(def env :dev)
(def params {})

(def cfg (infra.core/read-config "demo/aws-config.edn" env params))

(defn get-stack [k]
  (get cfg (infra/eid k env)))

(comment 
 (infra.aws/create-stack (get-stack "Stack1"))
 (infra.aws/create-stack (get-stack "Stack2"))
 )
```

List of available operations: 

* `create-stack` 
* `delete-stack` 

## References

Infra is built on Clojure's [edn](https://github.com/edn-format/edn) and Cognitecht's [aws-api](https://github.com/cognitect-labs/aws-api).

Infra is inspired by [aero](https://github.com/juxt/aero), which we originally used but since AWS already provides templating functions we decided to use EDN directly and have most custom reader literals match the AWS templating spec.

## Acknowledgments

Thanks to Joe Lane for discussing Code-As-Infrastructure with me and convincing me not to use Pulumi+Typescript.

## License

MIT © [Alid Lorenzo](https://github.com/alidlo)
