```powershell

  $definition = "https://binaries.avivagroup.com/artifactory/database-resources-generic-local/perfmon/collectors/SQLServer2016/definition.xml"
  $name =       "Database Performance"
  $path =       "S:/Data_X/diag_01/database"
  $duration =    9    ##  unit hours.
  $size =        900  ##  unit MB.

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
        $Collector.SetXML( $( Invoke-WebRequest -UseBasicParsing $definition ).Content );

        Write-Information "Creating Schedule for $name DataCollectors."
        $Schedule           = $CollectorSet.Schedules.CreateSchedule();
        $Schedule.Days      = 127;
        $Schedule.StartDate = [DateTime]$(Get-Date -Format "dd MMM yyyy 16:00:00");
        $Schedule.StartTime = [DateTime]$(Get-Date -Format "dd MMM yyyy 16:00:00");

        Write-Host "Creating the $name Data Collector Set."
        $CollectorSet.DisplayName               = $name;
        $CollectorSet.Description               = "SQL Performance Trace";
        $CollectorSet.Segment                   = $true;
        $CollectorSet.Duration                  = $(3600 * $duration);
        $CollectorSet.SegmentMaxSize            = $size
        $CollectorSet.SubdirectoryFormat        = 3;
        $CollectorSet.SubdirectoryFormatPattern = "yyyyMMdd\_NNN";
        $CollectorSet.RootPath                  = $($path).Replace("/","\");

        ##  setup data collector set.
        $CollectorSet.Schedules.Add($Schedule)
        $CollectorSet.DataCollectors.Add($Collector);
        $null = $CollectorSet.Commit($name,$null,0x0003);
        $CollectorSet.Query($name,$null)
      }

      $CollectorSet.start($false)

    }
    Catch [System.Exception] {
      Write-Host "Error at line: $(($PSItem.InvocationInfo.line).Trim())"
      if ( $Error.Count > 0 ) {
        Write-Host $($Error[0].Exception.Message)
      }
      $PSCmdlet.ThrowTerminatingError($PSItem)
    }

```
