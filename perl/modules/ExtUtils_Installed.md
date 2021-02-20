# Пример использования ExtUtils::Installed

Два простых сценария, которые позволяют прояснить - что имеется у хостера из Perl-модулей.

**Пример 1**

```perl
#!/usr/local/bin/perl

use ExtUtils::Installed;
my $installed = ExtUtils::Installed->new();

foreach my $module ($installed->modules()){
    printf "Module: %s\t\tDir: %s\t\tVersion: %s\n", $module, 
      $installed->directories($module), $installed->version($module);
}
```

**Пример 2**

```perl
#!/usr/local/bin/perl

use ExtUtils::Installed;

if (not defined $ARGV[0]){
  &amp;help()
} elsif ($ARGV[0] eq "-l"){
  &amp;list_modules();
} elsif ($ARGV[0] eq "-m" &amp;&amp; defined $ARGV[1] &amp;&amp; $ARGV[1] =~/^[a-zA-Z0-9-]*$/i){
  &amp;info();
} else{
  &amp;help();
}
exit;

sub help {
  print "usage: perl $0 [options] [module_name]\n";
  print "Options are:\n
    -l: Print list of installed modules\n
    -m: Print information about modules: [module_name]\n\n";
}

sub info {
  my $installed = ExtUtils::Installed->new();
  my $module = $ARGV[1];

  print "INFO: $module\n";
  printf "Dir prog:\t %s\n",$installed->directories($module, "prog");
  printf "Dir doc:\t %s\n",$installed->directories($module,"doc");

  printf "Dir tree:\t %s\n\n",$installed->directory_tree($module, "prog");

  printf "Files prog:\t %s\n",$installed->files($module, "prog");
  printf "Files doc:\t %s\n",$installed->files($module,"doc");
  print "Validate:\t\n"; my (@validate) = $installed->validate($module);

  foreach my $val (@validate){
    printf "\t %s\n",$val;
  }
}

sub list_modules {
  my $installed = ExtUtils::Installed->new();
  foreach my $module ($installed->modules()){
    printf "Module: %s\t\tVersion: %s\n", $module, $installed->version($module);
  }
}
```


