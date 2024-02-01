
```go
  
func deleteOldPoint(deviceID int64, minVersion int64, deleteVersion int64, ddb *dynamodb.DynamoDB, pointTableName string) error {  
   querys := []*dynamodb.BatchWriteItemInput{}  
   input := getInput(pointTableName)  
  
   requests := []*dynamodb.WriteRequest{}  
   for v := minVersion; v <= deleteVersion; v++ {  
      req := &dynamodb.WriteRequest{  
         DeleteRequest: &dynamodb.DeleteRequest{  
            Key: map[string]*dynamodb.AttributeValue{  
               "did": {N: aws.String(strconv.FormatInt(deviceID, 10))},  
               "v":   {N: aws.String(strconv.FormatInt(v, 10))},  
            },         },      }  
      requests = append(requests, req)  
  
      // The BatchWriteItem operation puts or deletes multiple items in one or more tables.  
      // A single call to BatchWriteItem can transmit up to 16MB of data over the network,      // consisting of up to 25 item put or delete operations.      if len(input.RequestItems[pointTableName]) == 25 || v == deleteVersion {  
         input.RequestItems[pointTableName] = requests  
         querys = append(querys, &input)  
  
         requests = []*dynamodb.WriteRequest{}  
         input = getInput(pointTableName)  
      }   }  
   for i, input := range querys {  
      startVersion := input.RequestItems[pointTableName][0].DeleteRequest.Key["v"].N  
      endVersion := input.RequestItems[pointTableName][len(input.RequestItems[pointTableName])-1].DeleteRequest.Key["v"].N  
  
      _, err := ddb.BatchWriteItemWithContext(context.Background(), input)  
      logging.DefaultLogger().Infof("deleted old point, device id: %v, batch num: %v, start version: %v, end version: %v", deviceID, i, startVersion, endVersion)  
      if err != nil {  
         return err  
      }  
   }   return nil  
}
```

