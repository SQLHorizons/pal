```powershell

  $definition = "https://binaries.avivagroup.com/artifactory/database-resources-generic-local/perfmon/collectors/SQLServer2016/definition.xml"
  $name =       "Light Database Performance"
  $path =       "S:/Data_X/diag_01/perflogs/database/Performance"
  $duration =    3    ##  unit hours.
  $size =        100  ##  unit MB.



    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::TLS12

    Try {

      $CollectorSet = New-Object -COM Pla.DataCollectorSet

      Try {
        $CollectorSet.Query($name,$null)
      }
      Catch {
        Write-Host "Creating DataCollectors for database Performance."
        $Collector                       = $CollectorSet.DataCollectors.CreateDataCollector(0);
        $Collector.Name                  = "collector";
        $Collector.FileName              = "collection-";
        $Collector.FileNameFormat        = 0x0001;
        $Collector.FileNameFormatPattern = "yyyyMMddHHmmss\-N";
        $Collector.SampleInterval        = 5;
        $Collector.LogFileFormat         = 0x0003;

        $Collector.PerformanceCounters   = @(
          "\Processor(_Total)\% Processor Time",
          "\Process(sqlservr)\% Privileged Time",
          "\Process(sqlservr)\% Processor Time",
          "\LogicalDisk(*)\Avg. Disk sec/Read",
          "\LogicalDisk(*)\Avg. Disk sec/Write",
          "\Network Interface(*)\Bytes Received/sec",
          "\Network Interface(*)\Bytes Sent/sec",
          "\Memory\% Committed Bytes In Use",
          "\Memory\Available MBytes",
          "\SQLServer:Buffer Manager\Page life expectancy",
          "\SQLServer:Databases(_Total)\Log Bytes Flushed/sec"
        );

        Write-Host "Creating the $name Data Collector Set."
        $CollectorSet.DisplayName               = $name;
        $CollectorSet.Description               = "SQL Performance Trace";
        $CollectorSet.Segment                   = $false;
        $CollectorSet.Duration                  = $(3600 * $duration);
        $CollectorSet.SegmentMaxSize            = $size
        $CollectorSet.SubdirectoryFormat        = 3;
        $CollectorSet.SubdirectoryFormatPattern = "yyyyMMdd\_NNN";
        $CollectorSet.RootPath                  = $($path).Replace("/","\");

        ##  setup data collector set.
        $CollectorSet.DataCollectors.Add($Collector);
        $null = $CollectorSet.Commit($name,$null,0x0003);
        $CollectorSet.Query($name,$null)
      }

      $CollectorSet.start($true)

    }
    Catch [System.Exception] {
      Write-Host "Error at line: $(($PSItem.InvocationInfo.line).Trim())"
      if ( $Error.Count > 0 ) {
        Write-Host $($Error[0].Exception.Message)
      }
      $PSCmdlet.ThrowTerminatingError($PSItem)
    }

```