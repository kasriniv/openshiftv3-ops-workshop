# Limiting Number of Self-Provisioned Projects

In this lab you will learn how to limit how many projects a user can have. By default, an authenticated user can create as many projects as they want to. You can limit this in the master configuration file.

## Step 1

The number of self-provisioned projects that a given user can request can be limited with the `ProjectRequestLimit` setting in the `/etc/origin/master/master-config.yaml` file. As always, make a backup before making changes.

```
cp /etc/origin/master/master-config.yaml{,.bak-$(date +%F)}
```

Add this config. In the end, it should look something like this.

```
grep -A 13 admissionConfig /etc/origin/master/master-config.yaml
admissionConfig:
  pluginConfig:
    ProjectRequestLimit:
      configuration:
        apiVersion: v1
        kind: ProjectRequestLimitConfig
        limits:
        - selector:
            level: admin 
        - selector:
            level: advanced 
          maxProjects: 10
        - maxProjects: 5 
    BuildDefaults:
```

The above configuration sets a global limit of 5 projects per user while allowing 10 projects for users with a label of `level=advanced` and unlimited projects for users with a label of `level=admin`.

It breaks down like this:

* For selector `level=admin`, no `maxProjects` is specified. This means that users with this label will not be restrained
* For selector `level=advanced`, a maximum number of 10 projects will be allowed.
* For the third entry, no selector is specified. This means that it will be applied to any user that doesn’t satisfy the previous two rules. Because rules are evaluated in order, this rule should be specified last.

Restart the `atomic-openshift-master` service

## Step 2

Login to the webui as `user-1`...you should have about 3 projects (if you have not deleted any...it is okay if you did).

![image](images/3-projects.png)

Try to create more than 5 projects (name them anything you wish)...you will see an error when you try to create more than 5 projects.

![image](images/no-more-projects.png)


**NOTE:** This limit does not apply to projects you were ASSINGED; only projects you create.

## Step 3

The user `user-1` has no label associated with it. This means that this user gets the default "5" max projects. You can take a look at the lables with the following command

```
oc describe user user-1
Name:		user-1
Namespace:	<none>
Created:	3 days ago
Labels:		<none>
Annotations:	<none>
Identities:	Local Authentication:user-1
```

Label this user with `level=advanced` to be able to create 10 projects.

```
oc label user user-1  level=advanced
```

Verify that the label was applied

```
oc describe user user-1
Name:		user-1
Namespace:	<none>
Created:	3 days ago
Labels:		level=advanced
Annotations:	<none>
Identities:	Local Authentication:user-1
```

You should now be able to create 10 projects

## Conclusion

In this lab you learned how to limit how many projects a user can create. Furthermore, you learned how to leverage labels in order to tier access.
