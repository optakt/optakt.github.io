    // server.go
    func (s *Server) GetHeightForBlock(_ context.Context, req *GetHeightForBlockRequest) (*GetHeightForBlockResponse, error) {

        err := s.validate.Struct(req)
        if err != nil {
            return nil, fmt.Errorf("bad request: %w", err)
        }

        blockID := flow.HashToID(req.BlockID)
        height, err := s.index.HeightForBlock(blockID)
        if err != nil {
            return nil, fmt.Errorf("could not get height for block: %w", err)
        }

        res := GetHeightForBlockResponse{
            BlockID: req.BlockID,
            Height:  height,
        }

        return &res, nil
    }