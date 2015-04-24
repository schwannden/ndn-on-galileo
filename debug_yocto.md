# Debug Yocto

Upon executing
```
bitbake image-full-galileo
```
Many things could happen.... But you use `Debian` and follow the steps I suggested, the errors should be in the following category.

#### Error on bitbake's logic

This happens only once for me, and is due to network instability. For example, `bitbake` can not fetch certain package A (see next error), and other package B depends on A. Instead outputting error message "error in A's do_fetch procedure", it output error on B, because B doesn't have all its required packages.

**Solution:** Re-execute `bitbake image-full-galileo` and things might get back to normal.

#### Error during do_fetch

You will see something like this `task 3874 failed (meta-oe/meta-oe/recipes-support/opencv_s.4.3.bb, do_fetch)`
This means `bitbake` can not fetch the required package. Perhaps because your internet is down or unstable, or because the repository is really down.

**Solution: ** Find out which recipe corresponds to the failure (in error message), which is `opencv_s.4.3.bb` in this case. Find a repository for the given package and provide the `SRC_URI` field with a valid link to that repository. For example, I found another repository `http://pkgs.fedoraproject.org/repo/pkgs/opencv/OpenCV-2.4.3.tar.bz2/c0a5af4ff9d0d540684c0bf00ef35dbe/OpenCv-2.4.3.tar.bz2`, so I changed the value of `SRC_URI` to that link.

Sometimes there is another check in the .bb file about the repository's check sum. Simply update `SRC_URI[md5sum]` or `SRC_URI[sha256sum]` to the new repository's checksum.

