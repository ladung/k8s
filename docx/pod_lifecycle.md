- ```Always``` means that the container will be restarted even if it exited with a zero exit code (i.e. successfully). This is useful when you don't care why the container exited, you just want to make sure that it is always running (e.g. a web server). This is the default.

- ```OnFailure``` means that the container will only be restarted if it exited with a non-zero exit code (i.e. something went wrong). This is useful when you want accomplish a certain task with the pod, and ensure that it completes successfully - if it doesn't it will be restarted until it does.

- ```Never``` means that the container will not be restarted regardless of why it exited.

```sh
These different restart policies basically map to the different controller types as you can see from kubectl run --help:

--restart="Always": The restart policy for this Pod. Legal values [Always, OnFailure, Never]. If set to 'Always' a deployment is created for this pod, if set to 'OnFailure', a job is created for this pod, if set to 'Never', a regular pod is created. For the latter two --replicas must be 1. Default 'Always'

And the pod user-guide:

ReplicationController is only appropriate for pods with RestartPolicy = Always. Job is only appropriate for pods with RestartPolicy equal to OnFailure or Never.
```

# Tham khảo:
- https://blog.vietnamlab.vn/kubernestes-best-paractice-p1/
- https://stackoverflow.com/questions/40530946/what-is-the-difference-between-always-and-on-failure-for-kubernetes-restart-poli