// server_internal_test.go
func TestServer_GetHeightForBlock(t *testing.T) {
    blockID := mocks.GenericHeader.ID()
    tests := []struct {
        name string
        req *GetHeightForBlockRequest
        mockErr error
        wantBlockID flow.Identifier
        checkErr require.ErrorAssertionFunc
    }{
        {
            name: "nominal case",
            req: &GetHeightForBlockRequest{
                BlockID: mocks.ByteSlice(blockID),
            },
            mockErr: nil, 
            wantBlockID: blockID,
            checkErr: require.NoError,
        },
        {
        	name: "handles invalid block request",
        	req: &GetHeightForBlockRequest{},
        	checkErr: require.Error,
        },
        {
        	name: "handles failure to retrieve block height from index",
        	req: &GetHeightForBlockRequest{
        		BlockID: mocks.ByteSlice(blockID),
        	},
        	mockErr: mocks.GenericError,
        	checkErr: require.Error,
        },
    }

    for _, test := range tests {
        test := test
        t.Run(test.name, func (t *testing.T) {
            t.Parallel()

            index := mocks.BaselineReader(t)
            index.HeightForBlockFunc = func(blockID flow.Identifier) (uint64, error) {
                return mocks.GenericHeight, test.mockErr
            }

            s := Server{
                index:    index,
                validate: validator.New(),
            }

            got, err := s.GetHeightForBlock(context.Background(), test.req)

            test.checkErr(t, err)
            if err == nil {
            	assert.Equal(t, mocks.GenericHeight, got.Height)
                assert.Equal(t, test.wantBlockID[:], got.BlockID)
            }
        })
    }
}