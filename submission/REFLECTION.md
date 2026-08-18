# Reflection

Anti-pattern dễ vướng nhất là xem lakehouse như "data dump có Parquet" rồi quên maintenance và lifecycle. Team thường tập trung ingest cho đủ dữ liệu, nhưng ít ai sở hữu các job sau đó: compaction, clustering, vacuum, orphan cleanup, checkpoint, và đồng bộ delete sang index phụ.

NB6 cho thấy vấn đề này rất cụ thể: compaction giảm file từ 200 xuống 11, clustering giúp point query chỉ mở 1/10 file, nhưng snapshot expiry/orphan cleanup vẫn là hai việc khác nhau. Nếu chỉ nói "đã expire" mà không quét orphan, storage bill có thể không giảm.

NB7 làm anti-pattern này nguy hiểm hơn trong AI/RAG: lakehouse đã xóa đúng, nhưng external vector index vẫn trả 8 tài liệu đã erase. Vì vậy production design phải coi vector DB là derived index có hợp đồng CDF/delete rõ ràng, không phải nguồn sự thật.
