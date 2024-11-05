1.Image Security: ECR, ECR public
Whenever we push image to ECR then amazon inspector scan image OS and programming language packages.

Image -> ECR -> Amazon inspector scans images -> send report to AWS security hub

Dockerfile:
Add user to Dockerfile and make it a non-privileged user
Don't use --privileged flag when running a container


2.User Access
