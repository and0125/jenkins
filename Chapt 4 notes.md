# Baking Machine Images with Packer

Instead of installing Jenkins after infrastructure is created, its best to package all changes into machine images. This way, any Jenkins upgrades or maintenance can happen in the image, and a new instance can be deployed, then the old image can be destroyed once the new image is working properly.

This process is called creating _immutable infrastructure_: the concept is that architecture components are re-created and replaced instead of continuously updated after they are created.

This book uses Packer, but you can also use Docker instead. Examples I'll do will follow along with Packer to understand the book first.

## Packer Templates

These template files are constructed with 3 main sections:

- variables: these ar evariables that can be overridden by Packer at runtime.
- builders
- provisioners
