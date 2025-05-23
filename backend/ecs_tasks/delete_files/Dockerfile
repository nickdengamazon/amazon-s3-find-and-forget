ARG src_path=backend/ecs_tasks/delete_files
ARG layers_path=backend/lambda_layers

FROM public.ecr.aws/docker/library/python:3.12.10-slim-bullseye AS base

RUN apt-get update --fix-missing && apt-get -y upgrade
RUN apt-get -y install g++ gcc libsnappy-dev

FROM base AS builder

ARG src_path
ARG layers_path

RUN mkdir /install
WORKDIR /install
COPY $src_path/requirements.txt /requirements.txt

RUN pip3 install \
    -r /requirements.txt \
    -t /install \
    --compile \
    --no-cache-dir

FROM base

ARG src_path
ARG layers_path

RUN groupadd -r s3f2 && useradd --no-log-init -r -m -g s3f2 s3f2
USER s3f2
RUN mkdir /home/s3f2/app
RUN echo ${src_path}
COPY --from=builder /install /home/s3f2/.local/lib/python3.12/site-packages/
WORKDIR /home/s3f2/app
COPY $src_path/* \
     $layers_path/boto_utils/python/boto_utils.py \
     /home/s3f2/app/

CMD ["python3", "-u", "main.py"]
