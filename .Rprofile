# -- BEGIN PACKRAT --
# Use private package library
local({
  # Create the private package library if it doesn't already exist
  appRoot <- normalizePath('.', winslash='/')
  libRoot <- file.path(appRoot, 'library')
  localLib <- file.path(libRoot, R.version$platform, getRversion())
  newLocalLib <- FALSE
  if (!file.exists(localLib)) {
    message('Creating private package library at ', localLib)
    dir.create(localLib, recursive=TRUE)
    newLocalLib <- TRUE
  }

  # If there's a new library (created to make changes to packages loaded in the
  # last R session), remove the old library and replace it with the new one. 
  newLibRoot <- file.path(appRoot, 'library.new', 'library')
  if (file.exists(newLibRoot)) {
    message('Applying Packrat library updates ... ', appendLF = FALSE)
    succeeded <- FALSE
    if (file.rename(libRoot, file.path(appRoot, 'library.old'))) {
      if (file.rename(newLibRoot, libRoot)) {
        succeeded <- TRUE
      } else {
        # Moved the old library out of the way but couldn't move the new 
        # in its place; move the old library back 
        file.rename(file.path(appRoot, 'library.old'), libRoot)
      }
    }
    if (succeeded) {
      message('OK')
    } else {
      message('FAILED')
      message("Packrat was not able to make changes to its local library at ", 
              localLib, ". Check this directory's permissions and run ",
              "packrat::restore() to try again.")
    }
  }
  
  # If the new library temporary folder exists, remove it now so we don't
  # attempt to reapply the same failed changes 
  if (file.exists(file.path(appRoot, 'library.new'))) {
    unlink(file.path(appRoot, 'library.new'), recursive = TRUE)
  }
  if (file.exists(file.path(appRoot, 'library.old'))) {
    unlink(file.path(appRoot, 'library.old'), recursive = TRUE)
  }
  
  .libPaths(localLib)

  # Bootstrap the installation of devtools and packrat
  if (newLocalLib) {
    if (length(find.package('packrat', lib.loc = localLib, quiet = TRUE)) == 0) {
      packratBootstrap <- new.env(parent=emptyenv())
      assign("initPackrat", function() {
        message("Initializing packrat and installing dependencies: ", 
                appendLF = FALSE)
        pkgs <- c("bitops", "stringr", "digest", "RCurl", "httr", "memoise",
                  "whisker", "evaluate", "devtools", "packrat")
        for (pkg in pkgs) {
          pkgPath <- file.path(appRoot, "packrat.sources", pkg)
          srcPkg <- file.path(pkgPath, list.files(pkgPath)[1])
          utils::install.packages(srcPkg, type = "source", repos = NULL, 
                                  verbose = FALSE, dependencies = FALSE, 
                                  quiet = TRUE)
          message(".", appendLF = FALSE)
        }
        message(".", appendLF = TRUE)
        packrat:::annotatePkgs(pkgs, appRoot)
        packrat::restore(appRoot)
        detach("packratBootstrap")
      }, envir = packratBootstrap)
      attach(packratBootstrap)
      message("Packrat needs to install the packages this project depends on. ", 
              "Run initPackrat() to get started.")
    }
  }

  # Remind the user to snapshot after making changes to the package library.
  addTaskCallback(function(expr, result, ok, printed) {
    if (length(expr) > 0 && 
        is(expr[[1]], "language")) {
      name <- deparse(expr[[1]])
      if (identical(name, "install.packages") || 
          identical(name, "remove.packages") ||
          identical(name, "update.packages")) {
        message("Changes made to this project's private library may need to ",
                "be snapshotted. Run packrat::status() to see ",
                "differences since the last snapshot.")
      }
    }
    return(TRUE)
  }) 
  
  invisible()
})
# -- END PACKRAT --
